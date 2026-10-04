# Class ``SpatialAudioRenderer3D``

Renders a 3D scene of [``SpatialAudioSource``](./SpatialAudioSource.md) instances relative to a single [``SpatialAudioListener3D``](./SpatialAudioListener3D.md).

Extends [``SpatialAudioRenderer``](./SpatialAudioRenderer.md), which provides the master channel, reverb, limiter, distance attenuation, lowpass and clustering.

Voices are panned with a ``PannerNode``. With the default ``"HRTF"`` panning model, sources also get front/back and elevation cues, which works best on headphones. The ``PannerNode`` only handles direction; distance attenuation is done by the renderer. HRTF panners are relatively expensive, but because far away sources are clustered, the number of panners stays low.

Uses a right-handed coordinate system with y up and -z forward, the same as Web Audio and three.js.

## Example

```ts
import { SpatialAudioRenderer3D, AudioClip } from "@fluex/fluexgl-dsp";

const renderer = new SpatialAudioRenderer3D(audioDevice, { panningModel: "HRTF" });

const source = renderer.createSource({ position: { x: 0, y: 2, z: -10 } });
source.attachAudioClip(clip);
clip.play();

renderer.start();
```

### Example: following a three.js camera

```ts
import * as THREE from "three";

const forward = new THREE.Vector3();

function animate() {
    camera.getWorldDirection(forward);

    renderer.listener
        .setPosition(camera.position.x, camera.position.y, camera.position.z)
        .setOrientation(forward, camera.up);

    spatialRenderer.update();
    webglRenderer.render(scene, camera);
    requestAnimationFrame(animate);
}
```

- - -

## Constructor

```ts
new SpatialAudioRenderer3D(target: AudioDevice | AudioContext, options?: Partial<SpatialAudioRenderer3DOptions>): SpatialAudioRenderer3D;
```

### Arguments
- ``target``: [``AudioDevice``](./AudioDevice.md) | ``AudioContext`` - Where the renderer's master channel is created. See [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#constructor).
- ``options?``: [``Partial<SpatialAudioRenderer3DOptions>``](../interfaces/SpatialAudioRenderer3DOptions.md) - Renderer configuration. Every field is optional.

### Defaults
The 3D renderer uses defaults that suit world units in meters:

| Option | Default |
|---|---|
| ``panningModel`` | ``"HRTF"`` |
| ``refDistance`` | ``1`` |
| ``maxDistance`` | ``100`` |
| ``clustering.splitDistance`` | ``15`` |
| ``clustering.mergeDistance`` | ``20`` |

All other defaults are listed in [``SpatialAudioRendererOptions``](../interfaces/SpatialAudioRendererOptions.md).

- - -

## Properties

All properties of [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#properties), plus:

### ``listener: SpatialAudioListener3D``
The [``SpatialAudioListener3D``](./SpatialAudioListener3D.md) of this renderer. Every renderer has exactly one listener.

- - -

## Methods

All methods of [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#methods), plus:

### ``getPanningModel(): SpatialPanningModel``
Returns the current panning model.

#### Arguments
No arguments

#### Returns
- [``SpatialPanningModel``](../interfaces/SpatialPanningModel.md) - ``"HRTF"``, ``"equalpower"`` or ``"stereo"``.

### ``setPanningModel(panningModel: SpatialPanningModel): SpatialAudioRenderer3D``
Changes how voices are panned. Existing voices are replaced with a short crossfade. Does nothing if the model is already active.

#### Arguments
- ``panningModel``: [``SpatialPanningModel``](../interfaces/SpatialPanningModel.md) - The new panning model.

#### Returns
- ``SpatialAudioRenderer3D`` - The same renderer.

### ``setSpatialAudioListener(listener: SpatialAudioListener3D): SpatialAudioRenderer3D``
Replaces the listener of this renderer.

#### Arguments
- ``listener``: [``SpatialAudioListener3D``](./SpatialAudioListener3D.md) - The new listener.

#### Returns
- ``SpatialAudioRenderer3D`` - The same renderer.

- - -

## Notes

### The renderer does not use ``AudioContext.listener``
The Web Audio listener is shared by everything on an ``AudioContext``. To allow multiple renderers on one context, this renderer converts every source into listener space itself and leaves ``AudioContext.listener`` untouched.

### Choosing a panning model
- ``"HRTF"``: front/back and elevation cues. Best on headphones, most expensive.
- ``"equalpower"``: cheaper, left/right only, but still through a ``PannerNode``.
- ``"stereo"``: uses a ``StereoPannerNode``, like the 2D renderer.

- - -

## Events

This class does not emit any events.

## Getters and setters

Inherited from [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#getters-and-setters) (``isRunning``).
