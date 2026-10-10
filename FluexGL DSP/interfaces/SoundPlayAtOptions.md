# SoundPlayAtOptions

Options for a positioned, fire-and-forget sound played with [``renderer.playAt()``](../classes/SpatialAudioRenderer.md).

```ts
interface SoundPlayAtOptions extends SoundPlayOptions, Omit<SpatialAudioSourceOptions, "position" | "volume" | "label"> {
    cull: boolean;
}
```

## About
Combines the options of the instance ([``SoundPlayOptions``](./SoundPlayOptions.md)) with the options of the spatial source it is played through ([``SpatialAudioSourceOptions``](./SpatialAudioSourceOptions.md)), except ``position`` (an argument of ``playAt()``), ``volume`` (taken from the instance) and ``label``. Every field is optional.

## Properties
- `volume`, `pitch`, `loop`, `offset`, `when` - See [``SoundPlayOptions``](./SoundPlayOptions.md).
- `bus`, `clusterable`, `reverbSendFactor`, `airAbsorption` - See [``SpatialAudioSourceOptions``](./SpatialAudioSourceOptions.md).
- `distanceModel`, `refDistance`, `maxDistance`, `rolloffFactor` - See [``SpatialAttenuationOptions``](./SpatialAttenuationOptions.md).
- `cull`: `boolean` - Whether a one-shot that is inaudible when it starts (too far away) is skipped entirely. No audio node is created for it, and ``playAt()`` returns ``null``. Looping sounds are never skipped, since the listener may come closer. Default ``true``.

## Example
```ts
// An explosion that can be heard from far away, on the effects bus.
renderer.playAt(explosion, position, {
    bus: effectsBus,
    refDistance: 20,
    maxDistance: 600,
    reverbSendFactor: 1.5
});

// A sound that should start even when it is too far away right now.
renderer.playAt(alarm, position, { cull: false });
```
