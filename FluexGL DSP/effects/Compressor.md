# Class ``Compressor``

A dynamic range compressor, built on the native Web Audio ``DynamicsCompressorNode``. Reduces the level of the signal above the threshold, after which a makeup gain can bring the overall level back up.

Extends [``Effector``](../classes/Effector.md). Unlike most effects, ``Compressor`` does not use an AudioWorklet, so it does not require WebAssembly and can be used before the DSP pipeline is initialized.

```
input -> DynamicsCompressorNode -> makeup gain -> output
```

## Example

```ts
import { Compressor } from "@fluex/fluexgl-dsp";

const compressor = new Compressor({
    threshold: -18,
    ratio: 4,
    attack: 0.005,
    release: 0.2,
    makeupGain: 4
});

channel.addEffect(compressor);

// Later, for example in a meter:
console.log(compressor.reduction); // e.g. -3.2 (dB)
```

- - -

## Constructor

```ts
new Compressor(options?: Partial<CompressorOptions>): Compressor;
```

### Arguments

- ``options?``: [``Partial<CompressorOptions>``](../interfaces/CompressorOptions.md)
  Optional compressor configuration. Every field is optional, falls back to the class defaults below, and is clamped to its valid range.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"Compressor"``.

### ``name: string``
Internal name for this effector. Defaults to ``"Compressor"``.

### ``threshold: number``
Level (dB) above which compression starts. Between ``-100`` and ``0``. Defaults to ``-24``.

### ``knee: number``
Range (dB) above the threshold over which the curve smoothly transitions to the full ratio. Between ``0`` and ``40``. Defaults to ``30``.

### ``ratio: number``
Amount of dB input change for 1 dB of output change. Between ``1`` and ``20``. Defaults to ``4``.

### ``attack: number``
Time (seconds) to reduce the gain by 10 dB. Between ``0`` and ``1``. Defaults to ``0.003``.

### ``release: number``
Time (seconds) to increase the gain by 10 dB. Between ``0`` and ``1``. Defaults to ``0.25``.

### ``makeupGain: number``
Gain (dB) applied after compression. Between ``-24`` and ``24``. Defaults to ``0``.

### ``compressorNode: DynamicsCompressorNode | null``
The native compressor node. ``null`` until the effect is attached.

### ``makeupGainNode: GainNode | null``
The gain node that applies ``makeupGain``. ``null`` until the effect is attached.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the ``DynamicsCompressorNode`` and makeup ``GainNode`` with the current option values. When the effect is attached again on the same ``AudioContext``, the existing nodes are reused.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to create the nodes.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): CompressorOptions``

Returns a plain object snapshot of the current option values.

#### Returns

- [``CompressorOptions``](../interfaces/CompressorOptions.md)

- - -

### ``setThreshold(threshold: number): boolean``
### ``setKnee(knee: number): boolean``
### ``setRatio(ratio: number): boolean``
### ``setAttack(attack: number): boolean``
### ``setRelease(release: number): boolean``
### ``setMakeupGain(makeupGain: number): boolean``

Update the corresponding option. The value is clamped to its valid range (see Properties) and applied with a short ramp (10 ms time constant) to avoid clicks.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean`` - ``true`` when the value was applied to the audio node. ``false`` when the effect is not attached yet; the value is stored and used once the effect is attached.

- - -

## Getters and setters

### ``get reduction(): number``
Current gain reduction in dB (``0`` or negative). Useful for metering. ``0`` while the effect is not attached.

### ``get inputNode(): AudioNode | null``
Returns ``compressorNode``. See [``Effector``](../classes/Effector.md#getters-and-setters).

### ``get outputNode(): AudioNode | null``
Returns ``makeupGainNode``. See [``Effector``](../classes/Effector.md#getters-and-setters).

- - -

## Events

``Compressor`` does not use an AudioWorklet processor, so it does not dispatch the processor events of [``Effector``](../classes/Effector.md).

- - -

## Notes

### Native or WebAssembly?
The native ``DynamicsCompressorNode`` runs in the browser's audio engine on the audio thread, costs very little, and its parameters change without clicks. A WebAssembly implementation would only add value for features the native node does not offer, such as a sidechain input, a configurable lookahead, or identical behavior across browsers.

### Automatic makeup gain
The Web Audio specification adds a small automatic makeup gain inside every ``DynamicsCompressorNode``, based on the threshold, knee and ratio. ``Compressor`` keeps this behavior. If you need a strict output ceiling, use [``Limiter``](./Limiter.md), which compensates for it.
