# SoundPlayOptions

Options for one instance of a [``Sound``](../classes/Sound.md).

```ts
interface SoundPlayOptions {
    volume: number;
    pitch: number;
    loop: boolean;
    offset: number;
    when: number;
}
```

## About
Used by [``sound.play()``](../classes/Sound.md) and [``source.play()``](../classes/SpatialAudioSource.md). Extended by [``SoundPlayAtOptions``](./SoundPlayAtOptions.md) for ``renderer.playAt()``. Every field is optional.

## Properties
- `volume`: `number` - Multiplied with the ``volume`` of the sound (and its random ``volumeVariation``). Default ``1``.
- `pitch`: `number` - Added to the ``pitch`` of the sound, in semitones. Default ``0``.
- `loop`: `boolean` - Whether this instance loops. Defaults to the ``loop`` option of the sound.
- `offset`: `number` - Offset into the sound, in seconds. Default ``0``.
- `when`: `number` - ``AudioContext`` time at which to start. Times in the past start immediately. Defaults to now.

## Example
```ts
// A heavier variant of the same gunshot.
gunshot.play(effectsBus, { volume: 1.2, pitch: -3 });
```
