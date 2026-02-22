# Class ``LowPassFilter``

A low-pass filter audio effector that attenuates frequencies above a configurable cutoff frequency.
It is commonly used to remove high-frequency content, smooth signals, or create filter sweeps.

Internally this effector is based on the Web Audio API ``BiquadFilterNode`` with ``lowpass`` type.

## Example

```ts
import { LowPassFilter } from "@fluex/fluexgl-dsp";

const lowPass = new LowPassFilter({
    frequency: 1200,
    q: 0.7
});

await lowPass.InitializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new LowPassFilter(options?: Partial<LowPassFilterOptions>): LowPassFilter;
```

### Arguments

- ``options?``: ``Partial<LowPassFilterOptions>``  
  Optional low-pass filter configuration options.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"LowPassFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"LowPassFilter"``.

### ``filterNode: BiquadFilterNode | null``
Underlying Web Audio API filter node. Available after initialization.

- - -

## Methods

### ``InitializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the low-pass filter by creating a ``BiquadFilterNode`` with ``type = "lowpass"`` and
applying the configured options.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the filter node.

#### Returns

- ``Promise<void>``

- - -

### ``SetOptions(options: Partial<LowPassFilterOptions>): void``

Applies filter parameters such as cutoff frequency and Q factor.
Live-updates the internal ``BiquadFilterNode`` when available.

#### Arguments

- ``options``: ``Partial<LowPassFilterOptions>``  
  Options to apply.

#### Returns

- ``void``
