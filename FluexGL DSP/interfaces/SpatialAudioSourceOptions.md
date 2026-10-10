# SpatialAudioSourceOptions

Spatial audio source configuration.

```ts
interface SpatialAudioSourceOptions extends SpatialAttenuationOptions {
    label: string | null;
    position: Vector2 | Vector3;
    volume: number;
    clusterable: boolean;
    reverbSendFactor: number;
    airAbsorption: boolean;
    bus: Channel | Master | null;
}
```

## About
Used by the [``SpatialAudioSource``](../classes/SpatialAudioSource.md) constructor, ``renderer.createSource()`` and (partly) ``renderer.playAt()``. Every field is optional. The fields of [``SpatialAttenuationOptions``](./SpatialAttenuationOptions.md) override the renderer's attenuation settings for this source only.

## Properties
- `label`: `string | null` - Custom label. Default ``null``.
- `position`: [`Vector2`](./Vector2.md) | [`Vector3`](./Vector3.md) - Initial position. ``z`` defaults to ``0``, and is ignored by the 2D renderer.
- `volume`: `number` - Volume before distance attenuation. Default ``1``.
- `clusterable`: `boolean` - Whether this source may share a voice with other sources when it is far away. Default ``true``.
- `reverbSendFactor`: `number` - Multiplier for the reverb send. ``0`` disables reverb for this source. Default ``1``.
- `airAbsorption`: `boolean` - Whether the distance based lowpass filter is applied. Default ``true``.
- `bus`: [`Channel`](../classes/Channel.md) | [`Master`](../classes/Master.md) | `null` - Bus the dry sound of this source ends up on, for example an "Entities" or "Ambience" channel. ``null`` uses the output of the renderer. Sources only share a voice with sources on the same bus. Default ``null``.
- `distanceModel`, `refDistance`, `maxDistance`, `rolloffFactor` - See [``SpatialAttenuationOptions``](./SpatialAttenuationOptions.md).
