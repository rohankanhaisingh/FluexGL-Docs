# Example 05: Spatial 3D with three.js

A first-person three.js scene where the listener follows the camera, and sounds are attached to meshes. Uses HRTF panning, so sounds can be heard in front, behind, above and below the player (best on headphones).

## What it shows
- [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md) with ``panningModel: "HRTF"``.
- Syncing [``SpatialAudioListener3D``](../classes/SpatialAudioListener3D.md) with a ``THREE.PerspectiveCamera``.
- Keeping sources in sync with ``THREE.Object3D`` world positions.
- Switching the panning model at runtime.

## Code

```ts
import * as THREE from "three";
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
import { AudioClip, AudioSourceData, DspPipeline, SpatialAudioRenderer3D, SpatialAudioSource, loadAudioSource } from "@fluex/fluexgl-dsp";

const webgl = new THREE.WebGLRenderer({ antialias: true });
webgl.setSize(innerWidth, innerHeight);
document.body.appendChild(webgl.domElement);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.05, 500);
camera.position.set(0, 1.7, 5);

const controls = new PointerLockControls(camera, document.body);

scene.add(new THREE.HemisphereLight("#ffffff", "#333333", 1));
scene.add(new THREE.GridHelper(200, 40));

let audio: SpatialAudioRenderer3D | null = null;

/** Sounds attached to meshes. The source follows the mesh every frame. */
const attachments: { object: THREE.Object3D, source: SpatialAudioSource }[] = [];

function attachSound(object: THREE.Object3D, data: AudioSourceData, loop: boolean = true): AudioClip {

    const position = object.getWorldPosition(new THREE.Vector3());
    const source = (audio as SpatialAudioRenderer3D).createSource({ position });
    const clip = new AudioClip(data);

    source.attachAudioClip(clip);
    clip.setLoop(loop);

    attachments.push({ object, source });
    return clip;
}

async function start() {

    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    await audioDevice.context.resume();

    // World units are meters.
    audio = new SpatialAudioRenderer3D(audioDevice, {
        panningModel: "HRTF",
        refDistance: 2,
        maxDistance: 120,
        clustering: { splitDistance: 20, mergeDistance: 28 }
    });

    const radioData = await loadAudioSource("/audio/radio.ogg");

    if (!radioData) return;

    // A radio on top of a pillar: walk up to it and look up to hear the elevation.
    const radio = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.4, 0.3), new THREE.MeshStandardMaterial({ color: "orange" }));
    radio.position.set(0, 6, -15);
    scene.add(radio);

    attachSound(radio, radioData).play();
}

const forward = new THREE.Vector3();
const worldPosition = new THREE.Vector3();

function animate() {

    if (audio) {

        // Listener = camera.
        camera.getWorldDirection(forward);

        audio.listener
            .setPosition(camera.position.x, camera.position.y, camera.position.z)
            .setOrientation(forward, camera.up);

        // Sources = meshes.
        for (const { object, source } of attachments) {
            object.getWorldPosition(worldPosition);
            source.setPosition(worldPosition.x, worldPosition.y, worldPosition.z);
        }

        audio.update();
    }

    webgl.render(scene, camera);
    requestAnimationFrame(animate);
}

document.addEventListener("click", () => {
    if (!audio) start();
    controls.lock();
});

// Compare panning models: H cycles HRTF -> equalpower -> stereo.
document.addEventListener("keydown", (event) => {

    if (!audio || event.key.toLowerCase() !== "h") return;

    const models = ["HRTF", "equalpower", "stereo"] as const;
    const next = models[(models.indexOf(audio.getPanningModel()) + 1) % models.length];

    audio.setPanningModel(next);
});

animate();
```

## Notes
- The renderer never touches ``AudioContext.listener``. Every renderer converts positions into its own listener space, so multiple renderers can share one ``AudioContext``.
- Without three.js, use ``setYawPitch(yaw, pitch)`` or ``lookAt(x, y, z)`` on the listener.
- The coordinate system matches three.js (y up, -z forward), so positions and directions can be passed as they are. A ``THREE.Vector3`` satisfies the [``Vector3``](../interfaces/Vector3.md) interface.
- HRTF panners are more expensive than stereo panners. Clustering keeps their number low; check ``audio.voices.length``.
