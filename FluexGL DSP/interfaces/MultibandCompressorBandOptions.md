# MultibandCompressorBandOptions

Settings of one band of a [``MultibandCompressor``](../effects/MultibandCompressor.md).

```ts
type MultibandCompressorBandName = "low" | "mid" | "high";

interface MultibandCompressorBandOptions {
    threshold: number;
    knee: number;
    ratio: number;
    attack: number;
    release: number;
    makeupGain: number;
}
```

## Properties
- `threshold`: `number` - Level (dB) above which compression starts, ``-100`` to ``0``.
- `knee`: `number` - Range (dB) over which the curve transitions, ``0`` to ``40``.
- `ratio`: `number` - ``1`` to ``20``.
- `attack`: `number` - Seconds, ``0`` to ``1``.
- `release`: `number` - Seconds, ``0`` to ``1``.
- `makeupGain`: `number` - Gain (dB) after compression, ``-24`` to ``24``. The automatic makeup gain of the native compressor is undone, so this is the only gain added.

See [``MultibandCompressor``](../effects/MultibandCompressor.md#properties) for the defaults per band.
