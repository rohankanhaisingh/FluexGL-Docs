# SpatialAttenuationOptions

Distance attenuation settings.

```ts
interface SpatialAttenuationOptions {
    distanceModel: SpatialDistanceModel;
    refDistance: number;
    maxDistance: number;
    rolloffFactor: number;
}
```

## About
Shared by [``SpatialAudioRendererOptions``](./SpatialAudioRendererOptions.md) (defaults for all sources) and [``SpatialAudioSourceOptions``](./SpatialAudioSourceOptions.md) (overrides for one source). ``refDistance`` and ``maxDistance`` also define the normalized distance used for the lowpass filter and the reverb send.

## Properties
- `distanceModel`: [`SpatialDistanceModel`](./SpatialDistanceModel.md) - How the volume drops over distance. Default ``"inverse"``.
- `refDistance`: `number` - Distance at which the volume starts to drop. Within this distance the source plays at full volume. Default ``50`` (2D) or ``1`` (3D).
- `maxDistance`: `number` - Distance at which the source is inaudible. Default ``1500`` (2D) or ``100`` (3D).
- `rolloffFactor`: `number` - How fast the volume drops over distance. Default ``1``.
