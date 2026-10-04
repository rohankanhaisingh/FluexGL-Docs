# SpatialAudioRenderer3DOptions

3D spatial renderer configuration.

```ts
interface SpatialAudioRenderer3DOptions extends SpatialAudioRendererOptions {
    panningModel: SpatialPanningModel;
}
```

## About
Used by the [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md) constructor. Every field is optional. All fields of [``SpatialAudioRendererOptions``](./SpatialAudioRendererOptions.md) are available as well, with defaults suited to meters: ``refDistance: 1``, ``maxDistance: 100``, ``clustering.splitDistance: 15`` and ``clustering.mergeDistance: 20``.

## Properties
- `panningModel`: [`SpatialPanningModel`](./SpatialPanningModel.md) - How voices are panned. Default ``"HRTF"``.
