# StereoDelayOptions

StereoDelay configuration.

```ts
interface StereoDelayOptions {
    delayLeftMs: number;
    delayRightMs: number;
    feedback: number;
    mix: number;
    strictMode: StrictMode;
}
```

## About
Used by [``StereoDelay``](../effects/StereoDelay.md). Extended by [``AdvancedDelayOptions``](./AdvancedDelayOptions.md).

## Properties
- `delayLeftMs`: `number` - Delay time of the left channel (ms), between ``1`` and ``4000``. Default ``300``.
- `delayRightMs`: `number` - Delay time of the right channel (ms), between ``1`` and ``4000``. Default ``450``.
- `feedback`: `number` - Between ``0`` and ``0.98``. Default ``0.35``.
- `mix`: `number` - Between ``0`` and ``1``. Default ``0.35``.
- `strictMode`: [`StrictMode`](./StrictMode.md)
