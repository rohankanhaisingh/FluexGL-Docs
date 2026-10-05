# StereoMonoMode

How a StereoMono effect routes the left and right channel.

```ts
type StereoMonoMode = "stereo" | "mono" | "swap" | "left" | "right" | "left-to-both" | "right-to-both" | "mid" | "side";
```

## About
Used by [``StereoMonoOptions``](./StereoMonoOptions.md) and [``StereoMono.setMode()``](../effects/StereoMono.md). With ``M = (L + R) / 2`` and ``S = (L - R) / 2``:

- `"stereo"`: ``(L, R)``. Unchanged. Default.
- `"mono"`: ``(M, M)``.
- `"mid"`: ``(M, M)``. Same as ``"mono"``; the counterpart of ``"side"``.
- `"side"`: ``(S, -S)``. Only the stereo difference.
- `"swap"`: ``(R, L)``.
- `"left"`: ``(L, 0)``. Keeps the left side in place.
- `"right"`: ``(0, R)``. Keeps the right side in place.
- `"left-to-both"`: ``(L, L)``.
- `"right-to-both"`: ``(R, R)``.

``"left"`` + ``"right"`` and ``"mid"`` + ``"side"`` add up to the original signal, so a split with these modes can be merged back losslessly.
