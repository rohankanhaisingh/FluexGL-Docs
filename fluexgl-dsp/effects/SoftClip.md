# Class ``SoftClip``

A soft-clipping distortion effector that gradually rounds off the signal peaks once driven, producing a warmer, less harsh distortion character than [``HardClip``](./HardClip.md).

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { SoftClip } from "@fluex/fluexgl-dsp";

const softClip = new SoftClip({
    drive: 2,
    gain: 0.8
});

await softClip.initializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new SoftClip(options?: Partial<SoftClipOptions>): SoftClip;
```

### Arguments

- ``options?``: [``Partial<SoftClipOptions>``](../interfaces/SoftClipOptions.md)
  Optional soft-clip configuration (``drive``, ``gain``, ``strictMode``). Missing or non-finite fields fall back to the class defaults below; ``drive`` and ``gain`` are floored at ``0``. Defaults to ``{}`` when omitted entirely.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"SoftClip"``.

### ``name: string``
Internal name for this effector. Defaults to ``"SoftClip"``.

### ``drive: number``
Amount of drive applied before clipping. Defaults to ``1``.

### ``gain: number``
Output gain applied after clipping. Defaults to ``1``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the effect by creating the AudioWorklet processor node (``SoftClipProcessor``) with the current option values. Unlike most other effectors, this guards against re-creating the node or overwriting an already-set ``context`` if called more than once.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): SoftClipOptions``

Returns a plain object snapshot of the current ``drive``, ``gain`` and ``strictMode`` values.

#### Returns

- [``SoftClipOptions``](../interfaces/SoftClipOptions.md)

- - -

### ``setDrive(drive: number): boolean``

Updates ``drive`` (floored at ``0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``drive``: ``number``

#### Returns

- ``boolean``

- - -

### ``setGain(gain: number): boolean``

Updates ``gain`` (floored at ``0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``gain``: ``number``

#### Returns

- ``boolean``

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md) (``incoming-processor-message``, ``incoming-processor-warning``, ``incoming-processor-error``, ``processor-wasm-instantiated``).

## Getters and setters

This class does not define public getters or setters.
