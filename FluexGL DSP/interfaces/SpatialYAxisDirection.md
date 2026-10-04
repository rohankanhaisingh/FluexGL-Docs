# SpatialYAxisDirection

Direction of the y-axis on screen, for 2D spatial audio.

```ts
type SpatialYAxisDirection = "down" | "up";
```

## About
Used by [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) and [``SpatialAudioListener``](../classes/SpatialAudioListener.md) to know which direction is "up" on the screen.

- `"down"`: y grows downwards, like a canvas or the DOM. Default.
- `"up"`: y grows upwards, like math coordinates.
