# Class ``HighPassFilter``

A high-pass filter audio effector that attenuates frequencies below a configurable cutoff frequency.
It is commonly used to remove low-frequency rumble, DC offsets, or to thin out signals in a mix.

Internally this effector runs its filtering inside the ``HighPassFilterProcessor`` AudioWorklet processor.

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { HighPassFilter } from "@fluex/fluexgl-dsp";

const highPass = new HighPassFilter({
    cutoff: 200,
    q: 0.7
});

...

channel.addEffect(highPass);
```

- - -

## Constructor

```ts
new HighPassFilter(options?: Partial<HighPassFilterOptions>): HighPassFilter;
```

### Arguments

- ``options?``: ``Partial<HighPassFilterOptions>``
  Optional high-pass filter configuration options (``cutoff``, ``q``, ``strictMode``). Defaults to ``{ cutoff: 1000, q: 0.7, strictMode: StrictMode.Disabled }`` when omitted.

<!-- TODO: verify -- HighPassFilterOptions is not currently re-exported from the package's top-level index (only the HighPassFilter class itself is), so there is no linkable interfaces page for it yet. -->

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"HighPassFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"HighPassFilter"``.

### ``cutoff: number``
Cutoff frequency in Hz. Defaults to ``1000``.

### ``q: number``
Resonance/Q factor, clamped between ``0.0001`` and ``4`` by ``setQ()``. Defaults to ``0.7``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

### ``maxFrequency: number``
Upper bound applied to ``cutoff`` by ``setCutoff()``. Defaults to half of ``DEFAULT_SAMPLE_RATE`` (``22050``).

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the filter by creating the AudioWorklet processor node (``HighPassFilterProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): HighPassFilterOptions``

Returns a plain object snapshot of the current ``cutoff``, ``q``, ``strictMode`` and ``maxFrequency`` values.

#### Returns

- ``HighPassFilterOptions``

- - -

### ``setCutoff(cutoff?: number): boolean``

Updates ``cutoff``, clamped to at most ``maxFrequency`` and to the current context's sample rate, and forwards the change to the AudioWorklet processor.

#### Arguments

- ``cutoff?``: ``number`` - Defaults to ``1000`` when omitted.

#### Returns

- ``boolean`` - ``false`` if this effect has no ``context`` yet, otherwise the result of sending the message to the processor.

- - -

### ``setMaxFrequency(maxFrequency?: number): boolean``

Updates ``maxFrequency`` and forwards the change to the AudioWorklet processor. Throws if the given value exceeds half of the AudioContext's sample rate.

#### Arguments

- ``maxFrequency?``: ``number`` - Defaults to the current ``maxFrequency`` when omitted.

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
