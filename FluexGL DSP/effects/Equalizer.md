# Class ``Equalizer``

A parametric equalizer with up to eight bands, processed in series in WebAssembly (RBJ biquad filters).

Extends [``Effector``](../classes/Effector.md).

Without options it starts as a flat 5-band EQ:

| Band | Type | Frequency | Q |
|---|---|---|---|
| 0 | ``lowshelf`` | 80 Hz | 0.7071 |
| 1 | ``peaking`` | 250 Hz | 1 |
| 2 | ``peaking`` | 1000 Hz | 1 |
| 3 | ``peaking`` | 4000 Hz | 1 |
| 4 | ``highshelf`` | 10000 Hz | 0.7071 |

## Band types

| Type | Uses gain | Q means |
|---|---|---|
| ``"peaking"`` | Yes | Bandwidth (higher = narrower) |
| ``"lowshelf"`` / ``"highshelf"`` | Yes | Slope of the shelf (0.7071 = no overshoot) |
| ``"lowpass"`` / ``"highpass"`` | No | Resonance at the cutoff (0.7071 = Butterworth) |
| ``"notch"`` / ``"bandpass"`` | No | Bandwidth |

## Example

```ts
import { Equalizer } from "@fluex/fluexgl-dsp";

// Default 5-band EQ, with a few changes.
const eq = new Equalizer();

eq.setBand(0, { gain: 3 });                 // +3 dB low shelf at 80 Hz
eq.setBand(2, { frequency: 800, gain: -4 }); // -4 dB around 800 Hz

channel.addEffect(eq);

// Or define the bands yourself:
const telephone = new Equalizer({
    bands: [
        { type: "highpass", frequency: 300, q: 0.7071 },
        { type: "peaking", frequency: 1500, gain: 6, q: 1.5 },
        { type: "lowpass", frequency: 3400, q: 0.7071 }
    ]
});
```

- - -

## Constructor

```ts
new Equalizer(options?: Partial<EqualizerOptions>): Equalizer;
```

### Arguments

- ``options?``: [``Partial<EqualizerOptions>``](../interfaces/EqualizerOptions.md)
  ``bands`` replaces the default bands (at most 8 are used). Missing band fields fall back to the default band at the same index.

- - -

## Properties

### ``static MAX_BANDS: number``
Maximum number of bands: ``8``.

### ``label: string | null``
Defaults to ``"Equalizer"``.

### ``name: string``
Defaults to ``"Equalizer"``.

### ``bands: EqualizerBand[]``
The current bands. Change them with ``setBand()``, ``addBand()`` or ``removeBand()``, not directly, so the processor is updated. See [``EqualizerBand``](../interfaces/EqualizerBand.md).

### ``outputGain: number``
Gain (dB) after the bands, between ``-24`` and ``24``. Defaults to ``0``.

### ``strictMode: StrictMode``
Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the ``EqualizerProcessor`` AudioWorklet node. Because ``parameterData`` only accepts numbers, the bands are passed as flattened keys (``band0Type``, ``band0Frequency``, ...).

### ``returnOptionsAsObject(): EqualizerOptions``

Returns a snapshot of the bands and output gain.

### ``getBand(index: number): EqualizerBand | null``

Returns a copy of a band, or ``null`` when it does not exist.

### ``setBand(index: number, band: Partial<EqualizerBand>): boolean``

Changes an existing band. Only the given fields change. Values are clamped: frequency ``10`` to ``24000`` Hz, gain ``-24`` to ``24`` dB, Q ``0.1`` to ``24``. The filter state is kept, so changing a band while audio plays does not click. Returns ``false`` when the index does not exist.

#### Arguments

- ``index: number, band: Partial<EqualizerBand>``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``addBand(band: Partial<EqualizerBand>): number``

Adds a band at the end (defaults: ``peaking``, 1000 Hz, 0 dB, Q 1, enabled). Returns its index, or ``-1`` when all eight bands are in use.

### ``removeBand(index: number): boolean``

Removes a band. The bands after it move up one position.

### ``flatten(): void``

Sets the gain of every band to 0 dB.

### ``setOutputGain(outputGain: number): boolean``

Sets the gain after the bands (dB), clamped between ``-24`` and ``24``.

#### Arguments

- ``outputGain: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md). The ``EqualizerProcessor`` processor sends ``processor-wasm-instantiated`` when its WebAssembly module is ready, and an ``incoming-processor-message`` for every parameter change.

## Requirements

Runs on an AudioWorklet backed by WebAssembly. Attach it only after ``await pipeline.initializeDpsPipeline()``, on the ``AudioDevice`` returned by ``resolveDefaultAudioOutputDevice()``. Requires a worklet and WASM build from FluexGL-DSP-WebAssembly 0.4.9 or newer, which registers ``EqualizerProcessor``.
