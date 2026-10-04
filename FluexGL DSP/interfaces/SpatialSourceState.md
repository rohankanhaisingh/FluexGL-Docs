# SpatialSourceState

Position of a spatial source relative to the listener.

```ts
interface SpatialSourceState {
    local: Vector3;
    distance: number;
    azimuth: number;
    elevation: number;
    attenuation: number;
    normalizedDistance: number;
}
```

## About
Calculated by ``renderer.computeSourceState()`` and stored in ``source.state`` on every ``renderer.update()``. Useful for debugging, UI and gameplay logic (for example: "is this sound audible?").

## Properties
- `local`: [`Vector3`](./Vector3.md) - Position of the source in listener space: ``x`` = right, ``y`` = up, ``z`` = forward. ``y`` is always ``0`` in 2D.
- `distance`: `number` - Distance to the listener.
- `azimuth`: `number` - Horizontal angle relative to the facing direction of the listener, in radians. Positive is to the right, ``Math.PI`` (or ``-Math.PI``) is directly behind.
- `elevation`: `number` - Vertical angle relative to the listener, in radians. Positive is up. Always ``0`` in 2D.
- `attenuation`: `number` - Distance based attenuation, between ``0`` and ``1``.
- `normalizedDistance`: `number` - Distance mapped from ``refDistance`` (``0``) to ``maxDistance`` (``1``).
