# EqualizerBand

One band of an [``Equalizer``](../effects/Equalizer.md).

```ts
type EqualizerBandType = "peaking" | "lowshelf" | "highshelf" | "lowpass" | "highpass" | "notch" | "bandpass";

interface EqualizerBand {
    type: EqualizerBandType;
    frequency: number;
    gain: number;
    q: number;
    enabled: boolean;
}
```

## Properties
- `type`: `EqualizerBandType` - Filter type. See the band type table on [``Equalizer``](../effects/Equalizer.md#band-types).
- `frequency`: `number` - Center or corner frequency (Hz), between ``10`` and ``24000`` (limited to just below Nyquist).
- `gain`: `number` - Boost or cut (dB), between ``-24`` and ``24``. Only used by ``"peaking"``, ``"lowshelf"`` and ``"highshelf"``.
- `q`: `number` - Between ``0.1`` and ``24``. Bandwidth, resonance or slope, depending on the type.
- `enabled`: `boolean` - A disabled band is skipped.
