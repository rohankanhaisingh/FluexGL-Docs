# Class ``HardClip``

A hard-clipping distortion effector that abruptly clips the signal once it crosses a driven threshold, producing an aggressive, buzzy distortion character.

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { HardClip } from "@fluex/fluexgl-dsp";

const hardClip = new HardClip({
    drive: 2,
    gain: 0.8
});

...

channel.addEffect(hardClip);
```

- - -

## Constructor

```ts
new HardClip(options?: HardClipOptions): HardClip;
```

### Arguments

- ``options?``: [``HardClipOptions``](../interfaces/HardClipOptions.md)
  Optional hard-clip configuration (``drive``, ``gain``, ``strictMode``). Missing or non-finite fields fall back to the class defaults below; ``drive`` and ``gain`` are floored at ``0``.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"HardClip"``.

### ``name: string``
Internal name for this effector. Defaults to ``"HardClip"``.

### ``drive: number``
Amount of drive applied before clipping. Defaults to ``1``.

### ``gain: number``
Output gain applied after clipping. Defaults to ``1``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the effect by creating the AudioWorklet processor node (``HardClipProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): HardClipOptions``

Returns a plain object snapshot of the current ``drive``, ``gain`` and ``strictMode`` values.

#### Returns

- [``HardClipOptions``](../interfaces/HardClipOptions.md)

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
