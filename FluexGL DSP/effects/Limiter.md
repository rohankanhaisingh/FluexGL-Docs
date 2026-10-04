# Class ``Limiter``

A peak limiter that keeps the output around a ceiling, built on the native Web Audio ``DynamicsCompressorNode`` (hard knee, 20:1 ratio, 1 ms attack). Typically placed at the end of a [``Master``](../classes/Master.md) chain, so many sounds playing at once do not clip.

Extends [``Effector``](../classes/Effector.md). ``Limiter`` does not use an AudioWorklet, so it does not require WebAssembly.

Every [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) and [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md) attaches a ``Limiter`` to its master channel by default (see ``renderer.limiter``).

```
input -> input gain -> DynamicsCompressorNode -> makeup compensation -> output
```

## Example

```ts
import { Limiter } from "@fluex/fluexgl-dsp";

const limiter = new Limiter({ ceiling: -1, release: 0.1 });

audioDevice.getMasterChannel().attachEffect(limiter);
```

### Example: configuring the limiter of a spatial renderer

```ts
const renderer = new SpatialAudioRenderer3D(audioDevice, {
    limiter: { ceiling: -3, release: 0.15 }
});

renderer.limiter?.setInputGain(3);

// Or disable it:
const noLimiter = new SpatialAudioRenderer2D(audioDevice, { limiter: false });
```

- - -

## Constructor

```ts
new Limiter(options?: Partial<LimiterOptions>): Limiter;
```

### Arguments

- ``options?``: [``Partial<LimiterOptions>``](../interfaces/LimiterOptions.md)
  Optional limiter configuration. Every field is optional, falls back to the class defaults below, and is clamped to its valid range.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"Limiter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"Limiter"``.

### ``ceiling: number``
Maximum output level (dB). Between ``-24`` and ``0``. Defaults to ``-1``.

### ``release: number``
Time (seconds) for the limiter to recover. Between ``0.01`` and ``1``. Defaults to ``0.1``.

### ``inputGain: number``
Gain (dB) applied before limiting, to drive the signal into the limiter. Between ``-24`` and ``24``. Defaults to ``0``.

### ``inputGainNode: GainNode | null``
Applies ``inputGain``. ``null`` until the effect is attached.

### ``compressorNode: DynamicsCompressorNode | null``
The native compressor node that does the limiting. ``null`` until the effect is attached.

### ``outputGainNode: GainNode | null``
Compensates the automatic makeup gain of the compressor node (see Notes). ``null`` until the effect is attached.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the input gain, compressor and output gain nodes with the current option values. When the effect is attached again on the same ``AudioContext``, the existing nodes are reused.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to create the nodes.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): LimiterOptions``

Returns a plain object snapshot of the current ``ceiling``, ``release`` and ``inputGain`` values.

#### Returns

- [``LimiterOptions``](../interfaces/LimiterOptions.md)

- - -

### ``setCeiling(ceiling: number): boolean``

Updates ``ceiling`` (clamped between ``-24`` and ``0``) and the makeup compensation, with a short ramp.

#### Arguments

- ``ceiling``: ``number``

#### Returns

- ``boolean`` - ``true`` when applied to the audio nodes, ``false`` when the effect is not attached yet (the value is used once attached).

- - -

### ``setRelease(release: number): boolean``

Updates ``release`` (clamped between ``0.01`` and ``1``), with a short ramp.

#### Arguments

- ``release``: ``number``

#### Returns

- ``boolean`` - ``true`` when applied to the audio node, ``false`` when the effect is not attached yet.

- - -

### ``setInputGain(inputGain: number): boolean``

Updates ``inputGain`` (clamped between ``-24`` and ``24``), with a short ramp.

#### Arguments

- ``inputGain``: ``number``

#### Returns

- ``boolean`` - ``true`` when applied to the audio node, ``false`` when the effect is not attached yet.

- - -

## Getters and setters

### ``get reduction(): number``
Current gain reduction in dB (``0`` or negative). Useful for metering. ``0`` while the effect is not attached.

### ``get inputNode(): AudioNode | null``
Returns ``inputGainNode``. See [``Effector``](../classes/Effector.md#getters-and-setters).

### ``get outputNode(): AudioNode | null``
Returns ``outputGainNode``. See [``Effector``](../classes/Effector.md#getters-and-setters).

- - -

## Events

``Limiter`` does not use an AudioWorklet processor, so it does not dispatch the processor events of [``Effector``](../classes/Effector.md).

- - -

## Notes

### Makeup compensation
The Web Audio specification adds an automatic makeup gain inside every ``DynamicsCompressorNode``: ``(1 / fullRangeGain) ^ 0.6``, where ``fullRangeGain`` is the curve's gain for a 0 dBFS input. For a ceiling of -1 dB this is about +0.57 dB, which would push the output above the ceiling. ``Limiter`` undoes this with ``outputGainNode``.

### Not a true-peak brickwall limiter
``Limiter`` is a safety limiter. The native node has a small fixed lookahead, so very fast transients can overshoot the ceiling by a fraction of a dB. A true-peak brickwall limiter with a configurable lookahead would require a WebAssembly implementation.

### Effect order on a master channel
[``Master``](../classes/Master.md) processes effects in the order they were attached, and does not offer a way to reorder them. Effects attached after the limiter come after it and can raise the level again.
