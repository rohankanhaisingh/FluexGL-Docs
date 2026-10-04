# SpatialAudioRenderer2DOptions

2D spatial renderer configuration.

```ts
interface SpatialAudioRenderer2DOptions extends SpatialAudioRendererOptions {
    yAxis: SpatialYAxisDirection;
}
```

## About
Used by the [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) constructor. Every field is optional. All fields of [``SpatialAudioRendererOptions``](./SpatialAudioRendererOptions.md) are available as well.

## Properties
- `yAxis`: [`SpatialYAxisDirection`](./SpatialYAxisDirection.md) - Direction of the y-axis on screen. Default ``"down"``.
