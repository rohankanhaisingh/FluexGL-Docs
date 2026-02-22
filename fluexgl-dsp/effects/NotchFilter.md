# Class ``NotchFilter``

A notch filter audio effector that attenuates a very narrow frequency band around a configurable
center frequency. It is commonly used to remove resonances, hum (e.g. 50/60 Hz), or feedback
frequencies without significantly affecting the rest of the spectrum.

Internally this effector is based on the Web Audio API ``BiquadFilterNode`` with ``notch`` type.

## Example

```ts
import { NotchFilter } from "@fluex/fluexgl-dsp";

const notch = new NotchFilter({
    frequency: 50,
    q: 10
});

await notch.InitializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new NotchFilter(options?: Partial<NotchFilterOptions>): NotchFilter;
```

### Arguments

- ``options?``: ``Partial<NotchFilterOptions>``  
  Optional notch filter configuration options.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"NotchFilter"``.

### ``name: string``
Internal name for this effector. Defaults to ``"NotchFilter"``.

### ``filterNode: BiquadFilterNode | null``
Underlying Web Audio API filter node. Available after initialization.

- - -

## Methods

### ``InitializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the notch filter by creating a ``BiquadFilterNode`` with ``type = "notch"`` and
applying the configured options.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the filter node.

#### Returns

- ``Promise<void>``

- - -

### ``SetOptions(options: Partial<NotchFilterOptions>): void``

Applies filter parameters such as center frequency and Q factor.
Live-updates the internal ``BiquadFilterNode`` when available.

#### Arguments

- ``options``: ``Partial<NotchFilterOptions>``  
  Options to apply.

#### Returns

- ``void``
