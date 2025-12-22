# ChorusEffectOptions

Chorus effect configuration.

```ts
interface ChorusEffectOptions {
    baseDelayMs: number;
    depthMs: number;
    rateHz: number;
    mix: number;
    feedback: number;
    strictMode: StrictMode;
}
```

## About
Defines parameters for the chorus DSP effect.

## Properties
- `baseDelayMs`: `number` - Base delay in milliseconds.
- `depthMs`: `number` - Modulation depth.
- `rateHz`: `number` - Modulation rate.
- `mix`: `number` - Dry/wet mix ratio.
- `feedback`: `number` - Feedback amount.
- `strictMode`: `StrictMode` - Validation behavior.

