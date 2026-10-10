# Class ``Sound``

A sound effect for games: one decoded ``AudioBuffer``, played as many lightweight [``SoundInstance``](./SoundInstance.md)s.

Unlike an [``AudioClip``](./AudioClip.md), a sound has no audio nodes of its own. Every instance is a single ``AudioBufferSourceNode`` (plus a ``GainNode`` when it does not have a spatial source to itself), which is released as soon as it has finished. This makes ``Sound`` the right choice for anything that is played often and from many places: gunshots, impacts, footsteps, explosions, collisions and ambience loops.

Per sound you can limit how many instances play at the same time (``maxInstances`` and ``steal``), skip starts that follow each other too quickly (``minInterval``), and add a random pitch and volume variation so repeated sounds do not sound mechanical. See [``SoundOptions``](../interfaces/SoundOptions.md).

A sound can be played in three ways:

| Call | Use it for |
|---|---|
| [``sound.play(bus)``](#methods) | Sounds that are not positioned: UI, music stingers, the player's own sounds. |
| [``source.play(sound)``](./SpatialAudioSource.md#methods) | Sounds of an object that has its own source, such as an NPC or a vehicle. The sound moves with the source. |
| [``renderer.playAt(sound, position)``](./SpatialAudioRenderer.md#methods) | Fire-and-forget sounds at a position that do not belong to an object: explosions, impacts, collisions, ambience of objects. |

## Example

```ts
import { Sound } from "@fluex/fluexgl-dsp";

const gunshot = await Sound.load(audioDevice, "/sfx/gunshot.wav", {
    maxInstances: 6,
    pitchVariation: 0.5
});

gunshot.play(effectsBus);                              // the player's own gun
renderer.playAt(gunshot, { x: 120, y: 0, z: -40 });    // another player's gun
```

- - -

## Constructor

```ts
new Sound(source: AudioBuffer | AudioSourceData, options?: Partial<SoundOptions>): Sound;
```

Creates a sound from an already decoded buffer, for example the result of [``loadAudioSource()``](../helpers/loadAudioSource.md). To load a file, ``Sound.load()`` is usually more convenient.

### Arguments
- ``source``: ``AudioBuffer`` | [``AudioSourceData``](../interfaces/AudioSourceData.md) - The decoded audio.
- ``options?``: [``Partial<SoundOptions>``](../interfaces/SoundOptions.md) - Limits and variation of this sound. Every field is optional.

- - -

## Static methods

### ``Sound.load(target: BaseAudioContext | { context: BaseAudioContext }, url: string, options?: Partial<SoundOptions>): Promise<Sound>``
Fetches and decodes an audio file with the given ``AudioContext`` or [``AudioDevice``](./AudioDevice.md). The decoded buffer is cached per context and url, so loading the same url again (also while the first load is still running) does not fetch or decode it again. A failed load is not cached, so it can be retried.

Decoding happens on the context the sound will be played on, so the buffer already has the right sample rate. [``loadAudioSource()``](../helpers/loadAudioSource.md) creates a temporary ``AudioContext`` for every file, which is noticeably slower when loading hundreds of sound effects.

#### Arguments
- ``target``: ``BaseAudioContext`` | [``AudioDevice``](./AudioDevice.md) - The context to decode with.
- ``url``: ``string`` - Url of the audio file.
- ``options?``: [``Partial<SoundOptions>``](../interfaces/SoundOptions.md) - ``label`` defaults to the url.

#### Returns
- ``Promise<Sound>``

#### Errors
- ``ERROR:FLUEXGL-DSP@0003`` (``ErrorCodes.PATH_TO_FILE_NOT_FOUND``) - The file could not be fetched or decoded. The promise rejects.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the sound. Should NOT be changed.

### ``buffer: AudioBuffer``
The decoded audio, shared by every instance.

### ``options: SoundOptions``
The resolved [``SoundOptions``](../interfaces/SoundOptions.md). Can be changed at runtime; changes apply to instances started afterwards.

- - -

## Methods

### ``play(target: Channel | Master, options?: Partial<SoundPlayOptions>): SoundInstance | null``
Plays the sound unpositioned on a bus [``Channel``](./Channel.md) or a [``Master``](./Master.md). The instance gets its own ``GainNode`` for its volume and fade-out.

#### Arguments
- ``target``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - Where the sound is played.
- ``options?``: [``Partial<SoundPlayOptions>``](../interfaces/SoundPlayOptions.md) - Volume, pitch, loop, offset and start time of this instance.

#### Returns
- [``SoundInstance``](./SoundInstance.md) | ``null`` - ``null`` when the start was skipped because of ``minInterval`` or ``maxInstances`` (with ``steal: "none"``, or ``"quietest"`` when the new instance is the quietest).

#### Errors
- ``ERROR:FLUEXGL-DSP@0014`` (``ErrorCodes.CHANNEL_NOT_INITIALIZED``) - The target has no ``context`` or ``input``, for example because it was disposed.

### ``stopAll(fadeTime?: number): void``
Stops every playing instance of this sound.

#### Arguments
- ``fadeTime?``: ``number`` - Fade-out in seconds. Defaults to ``0.02``.

#### Returns
- ``void``

### ``createInstance(target: SoundInstanceTarget): SoundInstance | null``
Starts an instance into the given destination, after checking ``minInterval`` and ``maxInstances``. Used internally by ``play()``, [``SpatialAudioSource.play()``](./SpatialAudioSource.md) and [``SpatialAudioRenderer.playAt()``](./SpatialAudioRenderer.md); prefer those. See [``SoundInstanceTarget``](../interfaces/SoundInstanceTarget.md).

#### Returns
- [``SoundInstance``](./SoundInstance.md) | ``null``

### ``remove(instance: SoundInstance): void``
Removes an instance from the list of playing instances, so it no longer counts towards ``maxInstances``. Called by the instance itself when it stops or ends; you do not need to call this.

- - -

## Getters and setters

### ``get duration(): number``
Duration of the buffer in seconds.

### ``get playing(): number``
Number of instances currently playing. Instances that are fading out after ``stop()`` no longer count.

- - -

## Instance limits

When ``play()`` is called while ``maxInstances`` instances are already playing, ``steal`` decides what happens:

| ``steal`` | Behavior |
|---|---|
| ``"oldest"`` (default) | The instance that started first is stopped with a 20 ms fade, and the new one starts. |
| ``"quietest"`` | The quietest instance is stopped, but only when the new instance is louder than it. Otherwise the new instance is skipped. For spatial instances, loudness includes the distance attenuation. |
| ``"none"`` | The new instance is skipped. |

``minInterval`` is checked first: a start within ``minInterval`` seconds after the previous start of the same sound is skipped. Use it for sounds that can be triggered many times in the same frame, such as collisions.

- - -

## Events

This class does not emit any events. Use [``instance.onEnded()``](./SoundInstance.md) to know when an instance has finished.

- - -

## Examples

### Example 1: loading a sound bank
```ts
const sounds = {
    gunshot: await Sound.load(audioDevice, "/sfx/gunshot.wav", { maxInstances: 6, pitchVariation: 0.5 }),
    impact: await Sound.load(audioDevice, "/sfx/impact.wav", { minInterval: 0.03, volumeVariation: 0.3 }),
    explosion: await Sound.load(audioDevice, "/sfx/explosion.wav", { maxInstances: 4, steal: "quietest" }),
    click: await Sound.load(audioDevice, "/sfx/click.wav", { maxInstances: 2 })
};
```

Or load them in parallel:

```ts
const [gunshot, impact] = await Promise.all([
    Sound.load(audioDevice, "/sfx/gunshot.wav", { maxInstances: 6 }),
    Sound.load(audioDevice, "/sfx/impact.wav", { minInterval: 0.03 })
]);
```

### Example 2: UI sounds
```ts
const ui = audioDevice.createChannel("UI");
ui.send(audioDevice.getMasterChannel());

button.addEventListener("click", () => sounds.click.play(ui));
```

### Example 3: a sound with a callback
```ts
sounds.explosion.play(effectsBus)?.onEnded(() => {
    console.log("explosion has finished");
});
```

See [Example 06: One-shot sounds](../examples/06-one-shot-sounds.md) and [Example 12: Game audio architecture](../examples/12-game-audio-architecture.md) for complete setups.
