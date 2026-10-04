# Class ``Reverb``

A reverb audio effector that simulates the reflections of a physical space, adding room size, damping and stereo spread to the dry signal.

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { Reverb } from "@fluex/fluexgl-dsp";

const reverb = new Reverb({
    roomSize: 0.5,
    damping: 0.4,
    mix: 0.35,
    stereoSpreadMs: 10
});

...

channel.addEffect(reverb);
```

- - -

## Constructor

```ts
new Reverb(options?: Partial<ReverbOptions>): Reverb;
```

### Arguments

- ``options?``: [``Partial<ReverbOptions>``](../interfaces/ReverbOptions.md)
  Optional reverb configuration. Every field is optional and falls back to the class defaults below; numeric fields are also floored at ``0``.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"Reverb"``.

### ``name: string``
Internal name for this effector. Defaults to ``"Reverb"``.

### ``roomSize: number``
Simulated room size. Defaults to ``0.3``.

### ``damping: number``
High-frequency damping amount. Defaults to ``0.5``.

### ``mix: number``
Dry/wet balance, between ``0`` (dry only) and ``1`` (wet only). Defaults to ``0.3``. The processor works with separate dry and wet levels; ``mix`` is sent as ``dry = 1 - mix`` and ``wet = mix``.

### ``stereoSpreadMs: number``
Stereo spread, in milliseconds. Defaults to ``0``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the reverb effect by creating the AudioWorklet processor node (``ReverbProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): ReverbOptions``

Returns a plain object snapshot of the current ``roomSize``, ``damping``, ``mix``, ``stereoSpreadMs`` and ``strictMode`` values.

#### Returns

- [``ReverbOptions``](../interfaces/ReverbOptions.md)

- - -

### ``setRoomSize(roomSize: number): boolean``

Updates ``roomSize`` (floored at ``0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``roomSize``: ``number``

#### Returns

- ``boolean``

- - -

### ``setDamping(damping: number): boolean``

Updates ``damping`` (floored at ``0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``damping``: ``number``

#### Returns

- ``boolean``

- - -

### ``setMix(mix: number): boolean``

Updates ``mix`` (clamped between ``0`` and ``1``) and sends ``dry = 1 - mix`` and ``wet = mix`` to the AudioWorklet processor.

> Before version 0.4.9, the command ids of this class did not match the ``ReverbProcessor``: ``setMix`` changed the dry level of the processor and ``setStereoSpreadMs`` its wet level, and the initial ``mix`` was ignored (the processor used dry 0.7 / wet 0.3). This has been fixed.

#### Arguments

- ``mix``: ``number``

#### Returns

- ``boolean``

- - -

### ``setStereoSpreadMs(stereoSpreadMs: number): boolean``

Updates ``stereoSpreadMs`` (floored at ``0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``stereoSpreadMs``: ``number``

#### Returns

- ``boolean``

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md) (``incoming-processor-message``, ``incoming-processor-warning``, ``incoming-processor-error``, ``processor-wasm-instantiated``).

## Getters and setters

This class does not define public getters or setters.
