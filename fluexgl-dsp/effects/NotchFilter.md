# Class ``NotchFilter``

A notch filter audio effector that attenuates a very narrow frequency band around a configurable
center frequency. It is commonly used to remove resonances, hum (e.g. 50/60 Hz), or feedback
frequencies without significantly affecting the rest of the spectrum.

Internally this effector runs its filtering inside the ``NotchFilterProcessor`` AudioWorklet processor.

Extends [``Effector``](../classes/Effector.md).

> **Not currently part of the public package API.** ``NotchFilter`` is a ``default`` export of its source file and is **not** re-exported from ``effects/exports.ts`` or from the package's top-level ``index.ts``. It cannot currently be imported as ``import { NotchFilter } from "@fluex/fluexgl-dsp"``. It is documented here for completeness since the class already exists in source, but treat it as unreleased/internal until it is added to the package's exports.
> <!-- TODO: verify -- confirm with the maintainer whether NotchFilter is intentionally withheld from the public exports or simply missing from effects/exports.ts. -->

## Example

```ts
// Not exported from "@fluex/fluexgl-dsp" yet - see note above.
import NotchFilter from "fluexgl-dsp/lib/src/effects/classes/filters/NotchFilter";

const notch = new NotchFilter({
    cutoff: 50,
    q: 10
});

await notch.initializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new NotchFilter(options?: Partial<NotchFilterOptions>): NotchFilter;
```

### Arguments

- ``options?``: ``Partial<NotchFilterOptions>``  
  Optional notch filter configuration options (``cutoff``, ``q``, ``minFrequency``, ``strictMode``).

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"NotchFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"NotchFilter"``.

### ``cutoff: number``
Center frequency in Hz. Defaults to ``1000``.

### ``minFrequency: number``
Lower bound applied to ``minFrequency`` by ``setMinFrequency()``. Defaults to ``10``.

### ``q: number``
Resonance/Q factor, clamped between ``0.0001`` and ``4`` by ``setQ()``. Defaults to ``0.7``.

``strictMode`` is tracked internally but, unlike the other filter effectors, is declared ``private`` on this class rather than ``public``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the filter by creating the AudioWorklet processor node (``NotchFilterProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): NotchFilterOptions``

Returns a plain object snapshot of the current ``cutoff``, ``q``, ``strictMode`` and ``minFrequency`` values.

#### Returns

- ``NotchFilterOptions``

- - -

### ``setCutoff(cutoff?: number): boolean``

Updates ``cutoff`` and forwards the change to the AudioWorklet processor.

<!-- TODO: verify -- setCutoff() clamps cutoff to Math.min(this.minFrequency, cutoff), which effectively caps the center frequency at minFrequency (10 by default). This looks like it may be a copy/paste bug carried over from HighPassFilter (which clamps against maxFrequency) rather than intended behavior, but it is documented here as-is because the source has not been changed. -->

#### Arguments

- ``cutoff?``: ``number`` - Defaults to ``1000`` when omitted.

#### Returns

- ``boolean`` - ``false`` if this effect has no ``context`` yet, otherwise the result of sending the message to the processor.

- - -

### ``setMinFrequency(minFrequency?: number): boolean``

Updates ``minFrequency`` (floored at ``10``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``minFrequency?``: ``number`` - Defaults to the current ``minFrequency`` when omitted.

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
