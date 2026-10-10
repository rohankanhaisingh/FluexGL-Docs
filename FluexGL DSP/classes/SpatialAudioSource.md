# Class ``SpatialAudioSource``

A positioned sound emitter in a 2D or 3D scene. The 2D renderer ignores the ``z`` coordinate.

[``Sound``](./Sound.md)s played with ``play()``, [``AudioClip``](./AudioClip.md) instances and [``Channel``](./Channel.md)s (for example an [``InputChannel``](./InputChannel.md) carrying a microphone or the voice of another player) attached to a source are routed through the source's own gain stage (volume and distance attenuation), and from there into a [``SpatialAudioVoice``](./SpatialAudioVoice.md) of the renderer. Depending on the distance to the listener, the source either has its own voice or shares one with other sources on the same bus (a cluster).

A source is cheap: two ``GainNode``s. Give every game object that makes sounds (an NPC, a vehicle, a door) its own source and play its sounds with ``play()``, instead of giving it a [``Channel``](./Channel.md). Use ``bus`` to route it to a bus channel. For sounds that do not belong to an object, use [``renderer.playAt()``](./SpatialAudioRenderer.md#methods) instead of creating a source.

## Example

```ts
import { AudioClip, SpatialAudioSource } from "@fluex/fluexgl-dsp";

// Usually created through the renderer:
const source = renderer.createSource({
    label: "waterfall",
    position: { x: 10, y: 0, z: -25 },
    volume: 0.8,
    refDistance: 5,
    bus: ambienceBus
});

source.play(waterfallLoop, { loop: true });

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

### ``bus: Channel | Master | null``
The bus [``Channel``](./Channel.md) or [``Master``](./Master.md) the dry sound of this source ends up on. ``null`` uses the output of the renderer. Can be changed at any time with ``setBus()``; the renderer moves the source to a voice on the new bus on its next ``update()``. Defaults to ``null``.

### ``attenuation: Partial<SpatialAttenuationOptions>``
Attenuation settings of this source. Missing values fall back to the renderer's settings. See [``SpatialAttenuationOptions``](../interfaces/SpatialAttenuationOptions.md).

### ``context: AudioContext | null``
The AudioContext of the renderer this source was added to. ``null`` until added.

### ``renderer: SpatialAudioRenderer | null``
The [``SpatialAudioRenderer``](./SpatialAudioRenderer.md) this source belongs to, or ``null``.

### ``audioClipPlayer: AudioClipPlayer | null``
The [``AudioClipPlayer``](./AudioClipPlayer.md) that plays the attached clips into ``input``. Only created once a clip is attached to an initialized source, so sources that only play [``Sound``](./Sound.md)s stay at two nodes. ``null`` until then.

### ``input: GainNode | null``
Receives the audio of all sounds, clips and channels.

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

### ``soundInstances: Set<SoundInstance>``
The [``SoundInstance``](./SoundInstance.md)s playing through this source. Maintained by the instances; should not be changed directly.

### ``virtualSince: number | null``
``AudioContext`` time since which this source has had no voice, or ``null``. Managed by the renderer for [loop virtualization](./SpatialAudioRenderer.md#loop-virtualization).

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

### ``setBus(bus: Channel | Master | null): SpatialAudioSource``
Routes this source to another bus. ``null`` uses the output of the renderer. Applied on the next renderer ``update()``.

#### Arguments
- ``bus``: [``Channel``](./Channel.md) | [``Master``](./Master.md) | ``null``

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``setAttenuation(attenuation: Partial<SpatialAttenuationOptions>): SpatialAudioSource``
Merges the given attenuation settings into ``attenuation``.

#### Arguments
- ``attenuation``: [``Partial<SpatialAttenuationOptions>``](../interfaces/SpatialAttenuationOptions.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``play(sound: Sound, options?: Partial<SoundPlayOptions>): SoundInstance | null``
Plays a [``Sound``](./Sound.md) through this source, so it is positioned at (and moves with) this source. Every instance gets its own volume; the volume of the source applies on top of it. Several sounds can play through one source at the same time. Looping instances are suspended automatically while the source has no voice (see [loop virtualization](./SpatialAudioRenderer.md#loop-virtualization)).

The source must be added to a renderer first.

#### Arguments
- ``sound``: [``Sound``](./Sound.md) - The sound to play.
- ``options?``: [``Partial<SoundPlayOptions>``](../interfaces/SoundPlayOptions.md) - Volume, pitch, loop, offset and start time of this instance.

#### Returns
- [``SoundInstance``](./SoundInstance.md) | ``null`` - ``null`` when the start was skipped because of the sound's ``minInterval`` or ``maxInstances``.

#### Errors
- ``ERROR:FLUEXGL-DSP@0014`` (``ErrorCodes.CHANNEL_NOT_INITIALIZED``) - The source has not been added to a renderer.

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

### ``attachChannel(channel: Channel): SpatialAudioSource``
Routes the output of a [``Channel``](./Channel.md) through this source, so it is positioned in the scene. Works with any channel, such as an [``InputChannel``](./InputChannel.md) carrying a microphone or the voice of another player (see [``InputChannel.setMediaStream()``](./InputChannel.md#methods)). Effects on the channel are applied before the spatialization. Can be called before the source is added to a renderer; the channel is then connected once the source is initialized. Attaching the same channel twice does nothing.

The channel should not be sent to a master channel as well, otherwise it is also heard unpositioned. Call ``channel.unsendFromAllMasters()`` first.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) - Must use the same ``AudioContext`` as the renderer.

#### Returns
- ``SpatialAudioSource`` - The same source.

#### Errors and warnings
- ``WARNING:FLUEXGL-DSP@0006`` (``WarningCodes.CHANNEL_ALSO_SENT_TO_MASTER``) - The channel is also sent to a master channel.
- ``ERROR:FLUEXGL-DSP@0015`` (``ErrorCodes.CHANNEL_NOT_SAME_AUDIO_CONTEXT``) - The channel and the renderer use a different ``AudioContext``.

### ``detachChannel(channel: Channel): SpatialAudioSource``
Stops routing the channel through this source. Detaching a channel that is not attached does nothing.

#### Arguments
- ``channel``: [``Channel``](./Channel.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``stopAll(): SpatialAudioSource``
Stops all attached clips and all sound instances playing through this source.

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

### ``reset(options?: Partial<SpatialAudioSourceOptions>): SpatialAudioSource``
Restores the default settings (position, volume, bus, attenuation, ...) and applies the given options, keeping the audio nodes. Used by the renderer to reuse pooled sources for ``playAt()``.

#### Arguments
- ``options?``: [``Partial<SpatialAudioSourceOptions>``](../interfaces/SpatialAudioSourceOptions.md)

#### Returns
- ``SpatialAudioSource`` - The same source.

### ``dispose(): void``
Stops all sound instances and clips, detaches all channels and releases the audio nodes. The detached channels themselves stay intact. Use ``renderer.removeSource(source)`` instead of calling this directly, so the source is first faded out of its voice.

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

### ``get channels(): Channel[]``
A copy of the list of attached channels (including channels waiting for the source to be initialized).

- - -

## Events

This class does not emit any events.

- - -

## Examples

### Example 1: a positioned microphone
```ts
const microphone = await audioDevice.createInputChannel(null, "Microphone");

const source = renderer.createSource({ position: { x: 300, y: 200 } });
source.attachChannel(microphone);

renderer.start();
```

### Example 2: a voice with a radio effect
```ts
import { HighPassFilter, LowPassFilter } from "@fluex/fluexgl-dsp";

const voice = new InputChannel(audioDevice.getContext(), "Player 2");
voice.setMediaStream(remoteStream);

// Effects run before the spatialization.
voice.addEffect(new HighPassFilter({ cutoff: 400 }));
voice.addEffect(new LowPassFilter({ cutoff: 3000 }));

renderer.createSource({ position: player2.position, clusterable: false }).attachChannel(voice);
```

See [Example 11: Proximity voice chat](../examples/11-proximity-voice-chat.md) for a complete setup.

### Example 3: an NPC with its own sounds
```ts
class Enemy {

    private source: SpatialAudioSource;
    private engine: SoundInstance | null;

    constructor(world: SpatialAudioRenderer3D, private sounds: GameSounds, bus: Channel) {
        this.source = world.createSource({ label: "enemy", bus });
        this.engine = this.source.play(sounds.engineLoop, { loop: true, volume: 0.4 });
    }

    public update(x: number, y: number, z: number) {
        this.source.setPosition(x, y, z);
    }

    public shoot() {
        this.source.play(this.sounds.gunshot);
    }

    public destroy(world: SpatialAudioRenderer3D) {
        // Fades the source out and stops its sounds, including the engine loop.
        world.removeSource(this.source);
    }
}
```
