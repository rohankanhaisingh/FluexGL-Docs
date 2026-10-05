# StereoSplitMode

How StereoMono.split() splits a channel.

```ts
type StereoSplitMode = "left-right" | "mid-side";
```

## About
Used by [``StereoMono.split()``](../effects/StereoMono.md).

- `"left-right"`: the first branch carries ``(L, 0)``, the second ``(0, R)``. Default.
- `"mid-side"`: the first branch carries the mid ``(M, M)``, the second the side ``(S, -S)``, with ``M = (L + R) / 2`` and ``S = (L - R) / 2``.

Both splits add up to the original signal when the branches are sent to the same channel.
