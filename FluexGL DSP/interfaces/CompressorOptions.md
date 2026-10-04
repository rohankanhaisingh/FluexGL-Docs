# CompressorOptions

Compressor configuration.

```ts
interface CompressorOptions {
    threshold: number;
    knee: number;
    ratio: number;
    attack: number;
    release: number;
    makeupGain: number;
}
```

## About
Defines parameters for the [``Compressor``](../effects/Compressor.md) effect. Values outside their range are clamped.

## Properties
- `threshold`: `number` - Level (dB) above which compression starts. Between ``-100`` and ``0``. Default ``-24``.
- `knee`: `number` - Range (dB) above the threshold over which the curve smoothly transitions. Between ``0`` and ``40``. Default ``30``.
- `ratio`: `number` - Amount of dB input change for 1 dB of output change. Between ``1`` and ``20``. Default ``4``.
- `attack`: `number` - Time (seconds) to reduce the gain by 10 dB. Between ``0`` and ``1``. Default ``0.003``.
- `release`: `number` - Time (seconds) to increase the gain by 10 dB. Between ``0`` and ``1``. Default ``0.25``.
- `makeupGain`: `number` - Gain (dB) applied after compression. Between ``-24`` and ``24``. Default ``0``.
