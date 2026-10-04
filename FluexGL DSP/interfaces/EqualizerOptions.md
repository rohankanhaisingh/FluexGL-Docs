# EqualizerOptions

Equalizer configuration.

```ts
interface EqualizerOptions {
    bands: Partial<EqualizerBand>[];
    outputGain: number;
    strictMode: StrictMode;
}
```

## About
Used by the [``Equalizer``](../effects/Equalizer.md) constructor.

## Properties
- `bands`: [`Partial<EqualizerBand>[]`](./EqualizerBand.md) - Up to 8 bands. Defaults to a flat 5-band layout. Missing fields fall back to the default band at the same index.
- `outputGain`: `number` - Gain (dB) after the bands, between ``-24`` and ``24``. Default ``0``.
- `strictMode`: [`StrictMode`](./StrictMode.md) - Default ``StrictMode.Disabled``.
