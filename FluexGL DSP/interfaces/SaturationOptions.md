# SaturationOptions

Saturation configuration.

```ts
type SaturationMode = "soft" | "tube" | "tape";

interface SaturationOptions {
    drive: number;
    mode: SaturationMode;
    tone: number;
    mix: number;
    outputGain: number;
    strictMode: StrictMode;
}
```

## About
Used by [``Saturation``](../effects/Saturation.md).

## Properties
- `drive`: `number` - How hard the signal is pushed into the curve (dB), ``0`` to ``48``. Default ``12``.
- `mode`: `SaturationMode` - ``"soft"``, ``"tube"`` or ``"tape"``. Default ``"soft"``.
- `tone`: `number` - Lowpass after the curve (Hz), ``0`` = off. Default ``0``.
- `mix`: `number` - Dry/wet mix, ``0`` to ``1``. Default ``1``.
- `outputGain`: `number` - Gain (dB) of the saturated signal, ``-24`` to ``24``. Default ``0``.
- `strictMode`: [`StrictMode`](./StrictMode.md)
