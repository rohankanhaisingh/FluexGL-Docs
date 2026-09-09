# Class ``LowPassFilter``

A low-pass filter audio effector that attenuates frequencies above a configurable cutoff frequency.
It is commonly used to remove high-frequency content, smooth signals, or create filter sweeps.

Internally this effector runs its filtering inside the ``LowPassFilterProcessor`` AudioWorklet processor.

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { LowPassFilter } from "@fluex/fluexgl-dsp";

const lowPass = new LowPassFilter({
    cutoff: 1200,
    q: 0.7
});

await lowPass.initializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new LowPassFilter(options: Partial<LowPassFilterOptions>): LowPassFilter;
```

### Arguments

- ``options``: [``Partial<LowPassFilterOptions>``](../interfaces/LowPassFilterOptions.md)
  Low-pass filter configuration options (``cutoff``, ``minFrequency``, ``q``, ``strictMode``). Falls back to the class defaults below for any field that is missing or not a finite number.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"LowPassFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"LowPassFilter"``.

### ``cutoff: number``
Cutoff frequency in Hz. Defaults to ``1000``.

### ``q: number``
Resonance/Q factor, clamped between ``0.0001`` and ``4`` by ``setQ()``. Defaults to ``0.7``.

### ``minFrequency: number``
Lower bound applied to ``cutoff`` by ``setCutoff()``. Defaults to ``10``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the filter by creating the AudioWorklet processor node (``LowPassFilterProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): LowPassFilterOptions``

Returns a plain object snapshot of the current ``cutoff``, ``minFrequency``, ``q`` and ``strictMode`` values.

#### Returns

- [``LowPassFilterOptions``](../interfaces/LowPassFilterOptions.md)

- - -

### ``setCutoff(cutoff?: number): boolean``

Updates ``cutoff``, clamped to at least ``minFrequency`` and to the current context's sample rate, and forwards the change to the AudioWorklet processor.

#### Arguments

- ``cutoff?``: ``number`` - Defaults to ``1000`` when omitted.

#### Returns

- ``boolean`` - ``false`` if this effect has no ``context`` yet, otherwise the result of sending the message to the processor.

- - -

### ``setMinFrequency(minFrequency?: number): boolean``

Updates ``minFrequency`` (floored at ``10``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``minFrequency?``: ``number`` - Defaults to ``10`` when omitted.

#### Returns

- ``boolean``

- - -

### ``setQ(q?: number): boolean``

Updates ``q``, clamped between ``0.0001`` and ``4``, and forwards the change to the AudioWorklet processor.

#### Arguments

- ``q?``: ``number`` - Defaults to ``0.7`` when omitted.

#### Returns

- ``boolean``

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md) (``incoming-processor-message``, ``incoming-processor-warning``, ``incoming-processor-error``, ``processor-wasm-instantiated``).

## Getters and setters

This class does not define public getters or setters.
