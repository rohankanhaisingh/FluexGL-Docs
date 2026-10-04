# SpatialClusteringOptions

Clustering configuration of a spatial renderer.

```ts
interface SpatialClusteringOptions {
    enabled: boolean;
    splitDistance: number;
    mergeDistance: number;
    maxAngle: number;
    maxDistanceRatio: number;
    maxMembers: number;
}
```

## About
Far away sources in roughly the same direction can share one voice (a cluster). When the listener comes close, the cluster is split and every source gets its own voice again. See [Clustering](../classes/SpatialAudioRenderer.md#clustering) for the exact algorithm.

The gap between ``splitDistance`` and ``mergeDistance`` prevents sources from flapping between a cluster and their own voice. If ``mergeDistance`` is not larger than ``splitDistance``, the renderer logs a warning and sets it to ``splitDistance * 1.25``.

Can be changed at runtime through ``renderer.clustering``.

## Properties
- `enabled`: `boolean` - Whether clustering is enabled. Default ``true``.
- `splitDistance`: `number` - Sources closer than this distance always get their own voice. Default ``300`` (2D) or ``15`` (3D).
- `mergeDistance`: `number` - Sources further than this distance may be merged into a cluster. Default ``400`` (2D) or ``20`` (3D).
- `maxAngle`: `number` - Maximum angle (radians) between a source and the cluster centre, as seen from the listener. Default ``Math.PI / 12`` (15 degrees).
- `maxDistanceRatio`: `number` - Maximum relative distance difference between a source and the cluster centre (``0.35`` = 35%). Default ``0.35``.
- `maxMembers`: `number` - Maximum amount of sources per cluster. Default ``16``.
