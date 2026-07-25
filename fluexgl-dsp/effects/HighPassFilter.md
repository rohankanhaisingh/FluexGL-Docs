# Class ``HighPassFilter``

A high-pass filter audio effector that attenuates frequencies below a configurable cutoff frequency.
It is commonly used to remove low-frequency rumble, DC offsets, or to thin out signals in a mix.

Internally this effector is based on the Web Audio API ``BiquadFilterNode`` with ``highpass`` type.

## Example

```ts
import { HighPassFilter } from "@fluex/fluexgl-dsp";

const highPass = new HighPassFilter({
    frequency: 200,
    q: 0.7
});

await highPass.InitializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new HighPassFilter(options?: Partial<HighPassFilterOptions>): HighPassFilter;
```

### Arguments

- ``options?``: ``Partial<HighPassFilterOptions>``  
  Optional high-pass filter configuration options.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"HighPassFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"HighPassFilter"``.

### ``filterNode: BiquadFilterNode | null``
Underlying Web Audio API filter node. Available after initialization.

- - -

## Methods

### ``InitializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the high-pass filter by creating a ``BiquadFilterNode`` with ``type = "highpass"`` and
applying the configured options.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the filter node.

#### Returns

- ``Promise<void>``

- - -

### ``SetOptions(options: Partial<HighPassFilterOptions>): void``

Applies filter parameters such as cutoff frequency and Q factor.
Live-updates the internal ``BiquadFilterNode`` when available.

#### Arguments

- ``options``: ``Partial<HighPassFilterOptions>``  
  Options to apply.

#### Returns

- ``void``
