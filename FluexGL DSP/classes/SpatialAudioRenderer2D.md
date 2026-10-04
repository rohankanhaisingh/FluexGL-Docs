# Class ``SpatialAudioRenderer2D``

Renders a 2D scene of [``SpatialAudioSource``](./SpatialAudioSource.md) instances relative to a single [``SpatialAudioListener``](./SpatialAudioListener.md). Voices are panned left and right with a ``StereoPannerNode``. The ``z`` coordinate of sources is ignored.

Extends [``SpatialAudioRenderer``](./SpatialAudioRenderer.md), which provides the master channel, reverb, limiter, distance attenuation, lowpass and clustering.

The listener rotation is optional. With the default rotation of ``0`` the listener faces up on the screen, so left and right on the screen are left and right in the stereo image. Games without a rotating listener do not have to set a rotation at all.

## Example

```ts
import { SpatialAudioRenderer2D, AudioClip, loadAudioSource } from "@fluex/fluexgl-dsp";

const renderer = new SpatialAudioRenderer2D(audioDevice, { maxDistance: 1600 });

const data = await loadAudioSource("/sounds/campfire.ogg");
const clip = new AudioClip(data);

const campfire = renderer.createSource({ position: { x: 400, y: 300 } });
campfire.attachAudioClip(clip);
clip.setLoop(true).play();

renderer.start();

// In your game loop:
renderer.listener.setPosition(player.x, player.y);
```

- - -

## Constructor

```ts
new SpatialAudioRenderer2D(target: AudioDevice | AudioContext, options?: Partial<SpatialAudioRenderer2DOptions>): SpatialAudioRenderer2D;
```

### Arguments
- ``target``: [``AudioDevice``](./AudioDevice.md) | ``AudioContext`` - Where the renderer's master channel is created. See [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#constructor).
- ``options?``: [``Partial<SpatialAudioRenderer2DOptions>``](../interfaces/SpatialAudioRenderer2DOptions.md) - Renderer configuration. Every field is optional.

### Defaults
| Option | Default |
|---|---|
| ``yAxis`` | ``"down"`` |
| ``refDistance`` | ``50`` |
| ``maxDistance`` | ``1500`` |
| ``clustering.splitDistance`` | ``300`` |
| ``clustering.mergeDistance`` | ``400`` |
| ``maxVoices`` | ``64`` |

All other defaults are listed in [``SpatialAudioRendererOptions``](../interfaces/SpatialAudioRendererOptions.md).

- - -

## Properties

All properties of [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#properties), plus:

### ``listener: SpatialAudioListener``
The [``SpatialAudioListener``](./SpatialAudioListener.md) of this renderer. Every renderer has exactly one listener. Its ``yAxis`` is kept in sync with the renderer.

### ``yAxis: SpatialYAxisDirection``
Direction of the y-axis on screen: ``"down"`` for canvas-like coordinates (default), ``"up"`` for math-like coordinates. See [``SpatialYAxisDirection``](../interfaces/SpatialYAxisDirection.md).

- - -

## Methods

All methods of [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#methods), plus:

### ``setSpatialAudioListener(listener: SpatialAudioListener): SpatialAudioRenderer2D``
Replaces the listener of this renderer. The listener's ``yAxis`` is set to the renderer's ``yAxis``.

#### Arguments
- ``listener``: [``SpatialAudioListener``](./SpatialAudioListener.md) - The new listener.

#### Returns
- ``SpatialAudioRenderer2D`` - The same renderer.

- - -

## Notes

### Side-scrollers
In a side-scroller, sources below the listener are not "behind" it. Set ``rearLowpassFactor: 1`` to disable the extra lowpass for sources behind the listener:

```ts
const renderer = new SpatialAudioRenderer2D(audioDevice, { rearLowpassFactor: 1 });
```

### Rotating listeners
For top-down games where the player rotates, update the rotation every frame, for example with ``lookAt()``:

```ts
renderer.listener.setPosition(player.x, player.y).lookAt(mouse.x, mouse.y);
```

- - -

## Events

This class does not emit any events.

## Getters and setters

Inherited from [``SpatialAudioRenderer``](./SpatialAudioRenderer.md#getters-and-setters) (``isRunning``).
