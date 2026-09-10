# LowPassFilterOptions

Low-pass filter configuration.

```ts
interface LowPassFilterOptions {
    cutoff: number;
    minFrequency: number;
    q: number;
    strictMode: StrictMode;
}
```

## About
Defines parameters for a low-pass filter.

## Properties
- `cutoff`: `number` - Cutoff frequency.
- `minFrequency`: `number` - Minimum allowed frequency.
- `q`: `number` - Resonance factor.
- `strictMode`: `StrictMode` - Validation behavior.

