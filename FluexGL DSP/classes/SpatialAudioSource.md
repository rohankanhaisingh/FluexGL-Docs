# Class ``SpatialAudioSource``

A positioned sound emitter in a 2D or 3D scene. The 2D renderer ignores the ``z`` coordinate.

[``AudioClip``](./AudioClip.md) instances attached to a source are routed through the source's own gain stage (volume and distance attenuation), and from there into a [``SpatialAudioVoice``](./SpatialAudioVoice.md) of the renderer. Depending on the distance to the listener, the source either has its own voice or shares one with other sources (a cluster).

## Example

```ts
import { AudioClip, SpatialAudioSource } from "@fluex/fluexgl-dsp";

// Usually created through the renderer:
const source = renderer.createSource({
    label: "waterfall",
    position: { x: 10, y: 0, z: -25 },
    volume: 0.8,
    refDistance: 5
});

const clip = new AudioClip(data);
source.attachAudioClip(clip);
clip.setLoop(true).play();

// Move it with your game object:
source.setPosition(entity.x, entity.y, entity.z);
```

- - -

## Constructor

```ts
new SpatialAudioSource(options?: Partial<SpatialAudioSourceOptions>): SpatialAudioSource;
```

A source has no audio nodes until it is added to a renderer with ``renderer.addSource()``. ``renderer.createSource()`` does both in one call.

### Arguments
- ``options?``: [``Partial<SpatialAudioSourceOptions>``](../interfaces/SpatialAudioSourceOptions.md) - Source configuration. The attenuation fields (``distanceModel``, ``refDistance``, ``maxDistance``, ``rolloffFactor``) override the renderer's settings for this source only.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the source. Should NOT be changed.

### ``label: string | null``
Custom label for this source. Defaults to ``null``.

### ``position: Vector3``
World position of the source. Defaults to ``{ x: 0, y: 0, z: 0 }``. Use ``setPosition()`` or change the fields directly.

### ``volume: number``
Volume of the source, before distance attenuation. Defaults to ``1``.

### ``clusterable: boolean``
Whether this source may share a voice with other sources when it is far away. Set to ``false`` for important sounds that should always keep their own voice. Defaults to ``true``.

### ``reverbSendFactor: number``
Multiplier for the reverb send of this source. ``0`` disables reverb for this source. Defaults to ``1``.

### ``airAbsorption: boolean``
Whether the distance based lowpass filter (and the extra lowpass for sources behind the listener) is applied. Defaults to ``true``.

### ``attenuation: Partial<SpatialAttenuationOptions>``
Attenuation settings of this source. Missing values fall back to the renderer's settings. See [``SpatialAttenuationOptions``](../interfaces/SpatialAttenuationOptions.md).

### ``context: AudioContext | null``
The AudioContext of the renderer this source was added to. ``null`` until added.

### ``renderer: SpatialAudioRenderer | null``
The [``SpatialAudioRenderer``](./SpatialAudioRenderer.md) this source belongs to, or ``null``.

### ``audioClipPlayer: AudioClipPlayer | null``
The [``AudioClipPlayer``](./AudioClipPlayer.md) that plays the attached clips into ``input``. ``null`` until added to a renderer.

### ``input: GainNode | null``
Receives the audio of all attached clips.

### ``output: GainNode | null``
Applies ``volume x attenuation``. Connected to a voice by the renderer.

### ``voice: SpatialAudioVoice | null``
The [``SpatialAudioVoice``](./SpatialAudioVoice.md) this source is currently rendered by. ``null`` when the source is inaudible. Managed by the renderer.

### ``state: SpatialSourceState | null``
Result of the last renderer ``update()``. ``null`` when the source has not been rendered yet. See [``SpatialSourceState``](../interfaces/SpatialSourceState.md).

### ``audible: boolean``
Whether the source is currently loud enough to be rendered. Managed by the renderer.

### ``renderedGain: number``
The gain (volume x attenuation) last sent to the audio thread. Managed by the renderer; also used to rank sources for the voice budget.

- - -

## Methods

### ``setPosition(x: number, y: number, z?: number): SpatialAudioSource``
Sets the world position. When ``z`` is omitted, the current ``z`` is kept.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``
- ``z?``: ``number`` - Defaults to the current ``position.z``.

#### Returns
- ``SpatialAudioSource`` - The same source. Can be used to stack methods.

### ``translate(dx: number, dy: number, dz?: number): SpatialAudioSource``
Moves the source by the given amount.

#### Arguments
- ``dx``: ``number``
- ``dy``: ``number``
- ``dz?``: ``number`` - Defaults to ``0``.

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``setVolume(volume: number): SpatialAudioSource``
Sets the volume (floored at ``0``). Applied on the next renderer ``update()``.

#### Arguments
- ``volume``: ``number``

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``setAttenuation(attenuation: Partial<SpatialAttenuationOptions>): SpatialAudioSource``
Merges the given attenuation settings into ``attenuation``.

#### Arguments
- ``attenuation``: [``Partial<SpatialAttenuationOptions>``](../interfaces/SpatialAttenuationOptions.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``attachAudioClip(audioClip: AudioClip): SpatialAudioSource``
Routes an [``AudioClip``](./AudioClip.md) through this source. Can be called before the source is added to a renderer; the clip is then attached once the source is initialized.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``detachAudioClip(audioClip: AudioClip): SpatialAudioSource``
Removes an [``AudioClip``](./AudioClip.md) from this source.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``stopAll(): SpatialAudioSource``
Stops all attached clips.

#### Arguments
No arguments

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``initialize(renderer: SpatialAudioRenderer, context: AudioContext): void``
Creates the audio nodes of this source. Called by the renderer when the source is added; you do not need to call this yourself. Logs an error when the source is already initialized on a different ``AudioContext``.

#### Arguments
- ``renderer``: [``SpatialAudioRenderer``](./SpatialAudioRenderer.md)
- ``context``: ``AudioContext``

#### Returns
- ``void``

### ``dispose(): void``
Stops all clips and releases the audio nodes. Use ``renderer.removeSource(source)`` instead of calling this directly, so the source is first faded out of its voice.

#### Arguments
No arguments

#### Returns
- ``void``

- - -

## Getters and setters

### ``get isVirtual(): boolean``
Whether the source is audible, but not rendered because the voice budget of the renderer (``maxVoices``) is used by louder sources. See [Voice budget](./SpatialAudioRenderer.md#voice-budget).

### ``get isInitialized(): boolean``
Whether the source has been added to a renderer and has its audio nodes.

### ``get audioClips(): AudioClip[]``
The attached clips (including clips waiting for the source to be initialized).

- - -

## Events

This class does not emit any events.
