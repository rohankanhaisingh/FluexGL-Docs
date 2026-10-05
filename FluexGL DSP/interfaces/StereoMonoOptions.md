# StereoMonoOptions

StereoMono configuration.

```ts
interface StereoMonoOptions {
    mode: StereoMonoMode;
    delayLeftMs: number;
    delayRightMs: number;
    invertLeft: boolean;
    invertRight: boolean;
}
```

## About
Used by [``StereoMono``](../effects/StereoMono.md).

## Properties
- `mode`: [`StereoMonoMode`](./StereoMonoMode.md) - How the left and right channel are routed. Default ``"stereo"`` (unchanged).
- `delayLeftMs`: `number` - Delay of the left output (ms), between ``0`` and ``100``. Default ``0``.
- `delayRightMs`: `number` - Delay of the right output (ms), between ``0`` and ``100``. Default ``0``.
- `invertLeft`: `boolean` - Inverts the polarity of the left output. Default ``false``.
- `invertRight`: `boolean` - Inverts the polarity of the right output. Default ``false``.
