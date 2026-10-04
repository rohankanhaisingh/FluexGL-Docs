# Example 03: Spatial 2D, top-down game

A top-down game where the player is the listener and turns towards the mouse. Enemies carry a looping sound, and a campfire plays at a fixed position.

## What it shows
- [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) with its own master channel.
- Sources that follow game objects.
- A rotating [``SpatialAudioListener``](../classes/SpatialAudioListener.md) (``lookAt``).
- Calling ``renderer.update()`` from your own game loop.

## Code

```ts
import { AudioClip, AudioSourceData, DspPipeline, SpatialAudioRenderer2D, SpatialAudioSource, loadAudioSource } from "@fluex/fluexgl-dsp";

interface Enemy {
    x: number;
    y: number;
    sound: SpatialAudioSource;
}

const canvas = document.querySelector("canvas") as HTMLCanvasElement;
const player = { x: 400, y: 300 };
const mouse = { x: 0, y: 0 };
const enemies: Enemy[] = [];

let renderer: SpatialAudioRenderer2D;

async function start() {

    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    await audioDevice.context.resume();

    // World units are pixels. Sounds are full volume within 60 px and silent at 1200 px.
    renderer = new SpatialAudioRenderer2D(audioDevice, {
        refDistance: 60,
        maxDistance: 1200,
        clustering: { splitDistance: 250, mergeDistance: 340 }
    });

    const [campfireData, droneData] = await Promise.all([
        loadAudioSource("/audio/campfire.ogg"),
        loadAudioSource("/audio/drone-hum.ogg")
    ]);

    if (!campfireData || !droneData) return;

    // A static sound.
    const campfire = renderer.createSource({ label: "campfire", position: { x: 900, y: 500 }, volume: 0.7 });
    const campfireClip = new AudioClip(campfireData);

    campfire.attachAudioClip(campfireClip);
    campfireClip.setLoop(true);
    campfireClip.play();

    // Moving sounds: one source per enemy. Every source needs its own AudioClip,
    // but the clips can share the same decoded AudioSourceData.
    for (let i = 0; i < 5; i++)
        spawnEnemy(droneData, Math.random() * 1600, Math.random() * 1200);

    requestAnimationFrame(loop);
}

function spawnEnemy(data: AudioSourceData, x: number, y: number) {

    const sound = renderer.createSource({ label: "enemy", position: { x, y } });
    const clip = new AudioClip(data);

    sound.attachAudioClip(clip);
    clip.setLoop(true);
    clip.play();

    enemies.push({ x, y, sound });
}

function killEnemy(enemy: Enemy) {

    // Fades the source out of its voice, then stops its clips and releases its nodes.
    renderer.removeSource(enemy.sound);
    enemies.splice(enemies.indexOf(enemy), 1);
}

function loop() {

    // ... move the player and enemies here ...

    // Listener = player. It turns towards the mouse, so sounds behind the player sound duller.
    renderer.listener.setPosition(player.x, player.y).lookAt(mouse.x, mouse.y);

    for (const enemy of enemies)
        enemy.sound.setPosition(enemy.x, enemy.y);

    renderer.update();

    requestAnimationFrame(loop);
}

canvas.addEventListener("mousemove", (event) => {
    mouse.x = event.offsetX;
    mouse.y = event.offsetY;
});

canvas.addEventListener("click", start, { once: true });
```

## Notes
- If your player does not rotate, skip ``lookAt``. With the default rotation of ``0``, the screen's left and right are the left and right speaker.
- ``renderer.start()`` runs ``update()`` on its own ``requestAnimationFrame`` loop. Prefer calling ``update()`` yourself after moving objects, so audio and graphics use the same positions.
- Use ``source.state`` to drive gameplay or UI, for example to show an indicator for enemies that can be heard: ``if (enemy.sound.audible) ...``.
- To remove all sounds when leaving a level, call ``renderer.dispose()``.
