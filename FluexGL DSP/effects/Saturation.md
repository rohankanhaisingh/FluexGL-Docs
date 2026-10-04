# Class ``Saturation``

Analog-style saturation, processed in WebAssembly with first-order antiderivative anti-aliasing (ADAA). ADAA strongly reduces the harsh, inharmonic aliasing of plain waveshaping, at a fraction of the cost of oversampling.

Extends [``Effector``](../classes/Effector.md).

## Modes

| Mode | Curve | Character |
|---|---|---|
| ``"soft"`` | ``tanh(x)`` | Smooth and symmetric. Odd harmonics. |
| ``"tube"`` | ``tanh(x + bias) - tanh(bias)`` | Asymmetric. Adds even harmonics, sounds warmer. A DC blocker removes the offset. |
| ``"tape"`` | ``x / (1 + abs(x))`` | Gentle knee. Compresses gradually. |

## Signal flow

```
input -+-> drive -> curve (ADAA) -> DC blocker -> tone (lowpass) -> 1/sqrt(drive) -> output gain -+-> mix -> output
       +----------------------------------------- dry --------------------------------------------+
```

The saturated signal is scaled by ``1 / sqrt(drive)``. That keeps level changes moderate when you raise the drive: quiet signals get somewhat louder, loud signals somewhat quieter. Use ``outputGain`` to match the level.

## Example

```ts
import { Saturation } from "@fluex/fluexgl-dsp";

const warmth = new Saturation({ mode: "tube", drive: 12, tone: 8000, mix: 0.5 });

channel.addEffect(warmth);

warmth.setDrive(18);
```

- - -

## Constructor

```ts
new Saturation(options?: Partial<SaturationOptions>): Saturation;
```

### Arguments

- ``options?``: [``Partial<SaturationOptions>``](../interfaces/SaturationOptions.md)
  Every field is optional, falls back to the defaults below, and is clamped to its valid range.

- - -

## Properties

| Property | Range | Default |
|---|---|---|
| ``drive: number`` | ``0`` to ``48`` dB | ``12`` |
| ``mode: SaturationMode`` | ``"soft"``, ``"tube"``, ``"tape"`` | ``"soft"`` |
| ``tone: number`` | ``0`` (off) or a lowpass frequency in Hz (from 200 Hz) | ``0`` |
| ``mix: number`` | ``0`` to ``1`` | ``1`` |
| ``outputGain: number`` | ``-24`` to ``24`` dB | ``0`` |
| ``strictMode: StrictMode`` | | ``StrictMode.Disabled`` |

``label`` and ``name`` default to ``"Saturation"``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the ``SaturationProcessor`` AudioWorklet node with the current values.

### ``returnOptionsAsObject(): SaturationOptions``

Returns a snapshot of the current values.

### ``setDrive(drive: number): boolean``

Sets the drive (dB), clamped between ``0`` and ``48``.

#### Arguments

- ``drive: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setMode(mode: SaturationMode): boolean``

Switches the curve. Returns ``false`` for an unknown mode.

#### Arguments

- ``mode: "soft" | "tube" | "tape"``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setTone(tone: number): boolean``

Sets the lowpass after the curve (Hz). ``0`` disables it.

#### Arguments

- ``tone: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setMix(mix: number): boolean``

Sets the dry/wet mix, clamped between ``0`` and ``1``.

#### Arguments

- ``mix: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setOutputGain(outputGain: number): boolean``

Sets the gain of the saturated signal (dB), clamped between ``-24`` and ``24``.

#### Arguments

- ``outputGain: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md). The ``SaturationProcessor`` processor sends ``processor-wasm-instantiated`` when its WebAssembly module is ready, and an ``incoming-processor-message`` for every parameter change.

## Requirements

Runs on an AudioWorklet backed by WebAssembly. Attach it only after ``await pipeline.initializeDpsPipeline()``, on the ``AudioDevice`` returned by ``resolveDefaultAudioOutputDevice()``. Requires a worklet and WASM build from FluexGL-DSP-WebAssembly 0.4.9 or newer, which registers ``SaturationProcessor``.
