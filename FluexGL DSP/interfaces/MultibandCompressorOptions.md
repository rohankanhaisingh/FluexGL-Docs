# MultibandCompressorOptions

MultibandCompressor configuration.

```ts
interface MultibandCompressorOptions {
    lowCrossover: number;
    highCrossover: number;
    low: Partial<MultibandCompressorBandOptions>;
    mid: Partial<MultibandCompressorBandOptions>;
    high: Partial<MultibandCompressorBandOptions>;
    outputGain: number;
}
```

## About
Used by the [``MultibandCompressor``](../effects/MultibandCompressor.md) constructor.

## Properties
- `lowCrossover`: `number` - Crossover between the low and mid band (Hz). Default ``200``.
- `highCrossover`: `number` - Crossover between the mid and high band (Hz), at least 1.5x ``lowCrossover``. Default ``2500``.
- `low`, `mid`, `high`: [`Partial<MultibandCompressorBandOptions>`](./MultibandCompressorBandOptions.md) - Settings per band.
- `outputGain`: `number` - Gain (dB) after the bands are summed, ``-24`` to ``24``. Default ``0``.
