# SpatialAudioListener3DOptions

3D spatial audio listener configuration.

```ts
interface SpatialAudioListener3DOptions {
    position: Vector3;
    forward: Vector3;
    up: Vector3;
}
```

## About
Used by the [``SpatialAudioListener3D``](../classes/SpatialAudioListener3D.md) constructor. Every field is optional. Directions do not have to be normalized.

## Properties
- `position`: [`Vector3`](./Vector3.md) - Initial position. Default ``{ x: 0, y: 0, z: 0 }``.
- `forward`: [`Vector3`](./Vector3.md) - Facing direction. Default ``{ x: 0, y: 0, z: -1 }``, like Web Audio and three.js.
- `up`: [`Vector3`](./Vector3.md) - Up direction. Default ``{ x: 0, y: 1, z: 0 }``.
