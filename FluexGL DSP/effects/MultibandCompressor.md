# Class ``MultibandCompressor``

A 3-band compressor, built from native Web Audio nodes, so it does not require WebAssembly. The signal is split into a low, mid and high band, every band is compressed separately, and the bands are summed again. This controls, for example, boomy lows without pumping the rest of the mix.

Extends [``Effector``](../classes/Effector.md).

## Signal flow

```
input -+-> LR4 lowpass (low x-over) -> allpass (high x-over) -> compressor -> makeup -+
       |                                                                              |
       +-> LR4 highpass (low x-over) -+-> LR4 lowpass (high x-over)  -> compressor -> makeup -+-> output gain -> output
                                      +-> LR4 highpass (high x-over) -> compressor -> makeup -+
```

- The crossovers are 4th-order **Linkwitz-Riley** filters (two cascaded Butterworth biquads).
- The low band passes through an **allpass** at the high crossover, so all three bands have the same phase. Without compression, the bands sum back to a perfectly flat response.
- Every native ``DynamicsCompressorNode`` adds an automatic makeup gain (about +8.5 dB per band at the default settings). ``MultibandCompressor`` **undoes** that per band, so ``makeupGain`` is the only gain that is added and quiet signals pass at their original level.

## Example

```ts
import { MultibandCompressor } from "@fluex/fluexgl-dsp";

const mbc = new MultibandCompressor({
    lowCrossover: 150,
    highCrossover: 3000,
    low: { threshold: -30, ratio: 4 },   // tame the low end
    mid: { threshold: -20, ratio: 2 },
    high: { threshold: -24, ratio: 3, makeupGain: 2 }
});

master.attachEffect(mbc);

// Metering
setInterval(() => console.log(mbc.reduction), 250); // { low: -4.2, mid: -0.8, high: -1.5 }
```

- - -

## Constructor

```ts
new MultibandCompressor(options?: Partial<MultibandCompressorOptions>): MultibandCompressor;
```

### Arguments

- ``options?``: [``Partial<MultibandCompressorOptions>``](../interfaces/MultibandCompressorOptions.md)
  Every field is optional. Band fields fall back to the band defaults below.

- - -

## Properties

### ``lowCrossover: number``
Crossover between the low and mid band (Hz). Defaults to ``200``.

### ``highCrossover: number``
Crossover between the mid and high band (Hz). Always at least 1.5x ``lowCrossover``. Defaults to ``2500``.

### ``outputGain: number``
Gain (dB) after the bands are summed, between ``-24`` and ``24``. Defaults to ``0``.

### ``bands: Record<"low" | "mid" | "high", MultibandCompressorBandOptions>``
The settings per band. Change them with ``setBand()``. Defaults:

| Band | threshold | knee | ratio | attack | release | makeupGain |
|---|---|---|---|---|---|---|
| ``low`` | -24 dB | 6 dB | 3 | 0.010 s | 0.20 s | 0 dB |
| ``mid`` | -24 dB | 6 dB | 3 | 0.005 s | 0.15 s | 0 dB |
| ``high`` | -24 dB | 6 dB | 3 | 0.002 s | 0.10 s | 0 dB |

### ``inputGainNode: GainNode | null`` / ``outputGainNode: GainNode | null``
Input and output nodes. ``null`` until attached.

``label`` and ``name`` default to ``"MultibandCompressor"``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the crossover filters, compressors and gain nodes.

### ``returnOptionsAsObject(): MultibandCompressorOptions``

Returns a snapshot of the crossovers, bands and output gain.

### ``setBand(name: "low" | "mid" | "high", options: Partial<MultibandCompressorBandOptions>): boolean``

Changes the settings of one band. Only the given fields change, with a 10 ms ramp. See [``MultibandCompressorBandOptions``](../interfaces/MultibandCompressorBandOptions.md) for the ranges.

#### Arguments

- ``name, options``

#### Returns

- ``boolean`` - ``true`` when applied to the audio nodes, ``false`` when the effect is not attached yet (the values are used on attachment).

### ``setCrossovers(lowCrossover: number, highCrossover: number): boolean``

Moves both crossovers (Hz). The low crossover is clamped between 20 Hz and 13.3 kHz; the high crossover between 1.5x the low crossover and 20 kHz.

#### Arguments

- ``lowCrossover: number, highCrossover: number``

#### Returns

- ``boolean`` - ``true`` when applied to the filters, ``false`` when the effect is not attached yet.

### ``setLowCrossover(frequency: number): boolean`` / ``setHighCrossover(frequency: number): boolean``

Moves one crossover. Same rules as ``setCrossovers()``.

### ``setOutputGain(outputGain: number): boolean``

Sets the gain after the bands are summed (dB).

#### Arguments

- ``outputGain: number``

#### Returns

- ``boolean`` - ``true`` when applied, ``false`` when the effect is not attached yet.

## Getters and setters

### ``get reduction(): { low: number, mid: number, high: number }``
Current gain reduction per band in dB (``0`` or negative). Useful for metering.

### ``get inputNode(): AudioNode | null`` / ``get outputNode(): AudioNode | null``
Return ``inputGainNode`` and ``outputGainNode``.

- - -

## Events

``MultibandCompressor`` does not use an AudioWorklet processor, so it does not dispatch processor events.
