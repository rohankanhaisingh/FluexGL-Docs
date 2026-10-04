# SpatialPanningModel

How the voices of a spatial renderer are panned.

```ts
type SpatialPanningModel = "stereo" | "equalpower" | "HRTF";
```

## About
Used by [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md). [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) always uses ``"stereo"``.

- `"stereo"`: a ``StereoPannerNode``. Left/right only, cheapest.
- `"equalpower"`: a ``PannerNode`` with the equal-power model. Left/right only.
- `"HRTF"`: a ``PannerNode`` with head-related transfer functions. Adds front/back and elevation cues. Best on headphones, most expensive. Default for 3D.
