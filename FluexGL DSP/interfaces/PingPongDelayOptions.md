# PingPongDelayOptions

PingPongDelay configuration.

```ts
interface PingPongDelayOptions extends MonoDelayOptions {}
```

## About
Used by [``PingPongDelay``](../effects/PingPongDelay.md). Same fields as [``MonoDelayOptions``](./MonoDelayOptions.md); the default ``feedback`` of ``PingPongDelay`` is ``0.5``.

## Properties
- `delayMs`: `number` - Time between two echoes (ms), between ``1`` and ``4000``. Default ``300``.
- `feedback`: `number` - Between ``0`` and ``0.98``. Default ``0.5``.
- `mix`: `number` - Between ``0`` and ``1``. Default ``0.35``.
- `strictMode`: [`StrictMode`](./StrictMode.md)
