# MonoDelayOptions

MonoDelay configuration.

```ts
interface MonoDelayOptions {
    delayMs: number;
    feedback: number;
    mix: number;
    strictMode: StrictMode;
}
```

## About
Used by [``MonoDelay``](../effects/MonoDelay.md). [``PingPongDelayOptions``](./PingPongDelayOptions.md) has the same fields.

## Properties
- `delayMs`: `number` - Delay time (ms), between ``1`` and ``4000``. Default ``300``.
- `feedback`: `number` - Between ``0`` and ``0.98``. Default ``0.35``.
- `mix`: `number` - Dry/wet mix, between ``0`` and ``1``. Default ``0.35``.
- `strictMode`: [`StrictMode`](./StrictMode.md) - Default ``StrictMode.Disabled``.
