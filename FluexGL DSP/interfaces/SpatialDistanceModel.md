# SpatialDistanceModel

Distance model used to calculate the volume of a spatial source.

```ts
type SpatialDistanceModel = "linear" | "inverse" | "exponential";
```

## About
Used by [``SpatialAttenuationOptions``](./SpatialAttenuationOptions.md). The formulas are the same as the distance models of the Web Audio ``PannerNode``, where ``d`` is the distance clamped between ``refDistance`` and ``maxDistance``:

- `"linear"`: ``1 - rolloffFactor * (d - refDistance) / (maxDistance - refDistance)`` (``rolloffFactor`` clamped between ``0`` and ``1``).
- `"inverse"`: ``refDistance / (refDistance + rolloffFactor * (d - refDistance))``. Default. Sounds the most natural.
- `"exponential"`: ``(d / refDistance) ^ -rolloffFactor``.

On top of that, the volume fades out with a smoothstep over the last 10% before ``maxDistance``, so every model is actually silent at ``maxDistance``.
