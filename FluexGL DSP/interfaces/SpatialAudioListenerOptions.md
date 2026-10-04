# SpatialAudioListenerOptions

2D spatial audio listener configuration.

```ts
interface SpatialAudioListenerOptions {
    position: Vector2;
    rotation: number;
    yAxis: SpatialYAxisDirection;
}
```

## About
Used by the [``SpatialAudioListener``](../classes/SpatialAudioListener.md) constructor. Every field is optional.

## Properties
- `position`: [`Vector2`](./Vector2.md) - Initial position. Default ``{ x: 0, y: 0 }``.
- `rotation`: `number` - Optional facing direction in radians, clockwise. ``0`` means facing up on the screen, so left and right map to the x-axis. Games without a rotating listener can ignore this. Default ``0``.
- `yAxis`: [`SpatialYAxisDirection`](./SpatialYAxisDirection.md) - Direction of the y-axis on screen. Set by the renderer. Default ``"down"``.
