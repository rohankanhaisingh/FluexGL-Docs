# SoundOptions

Limits and variation of a [``Sound``](../classes/Sound.md).

```ts
interface SoundOptions {
    label: string | null;
    volume: number;
    volumeVariation: number;
    pitch: number;
    pitchVariation: number;
    maxInstances: number;
    steal: SoundStealMode;
    minInterval: number;
    loop: boolean;
}
```

## About
Used by the [``Sound``](../classes/Sound.md) constructor and ``Sound.load()``. Every field is optional. Stored as ``sound.options`` and can be changed at runtime; changes apply to instances started afterwards.

## Properties
- `label`: `string | null` - Custom label. ``Sound.load()`` uses the url. Default ``null``.
- `volume`: `number` - Base volume of every instance. Default ``1``.
- `volumeVariation`: `number` - Random volume reduction per instance, between ``0`` (none) and ``1``. ``0.2`` plays every instance at 80% to 100% of the volume. Default ``0``.
- `pitch`: `number` - Base pitch in semitones. Changes the playback rate, so it also changes the duration. Default ``0``.
- `pitchVariation`: `number` - Random pitch offset per instance, in semitones (plus or minus). ``0.5`` to ``1`` makes repeated sounds such as footsteps and gunshots less mechanical. Default ``0``.
- `maxInstances`: `number` - Maximum number of instances of this sound playing at the same time. ``Infinity`` disables the limit. Default ``8``.
- `steal`: [`SoundStealMode`](./SoundStealMode.md) - What happens when ``maxInstances`` is reached. Default ``"oldest"``.
- `minInterval`: `number` - Minimum time in seconds between two starts of this sound. Starts within this window are skipped, for example when 40 collisions happen in the same frame. Default ``0``.
- `loop`: `boolean` - Whether instances loop by default. Can be overridden per instance. Default ``false``.

## Example
```ts
const footstep = await Sound.load(audioDevice, "/sfx/step.wav", {
    volume: 0.6,
    volumeVariation: 0.25,
    pitchVariation: 1,
    maxInstances: 4
});
```
