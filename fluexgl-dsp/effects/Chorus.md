# Class ``Chorus``

A chorus audio effector that creates a thickened, widened sound by duplicating the signal,
modulating delay times, and mixing the result back with the dry signal.

This effector is typically used for pads, leads, and guitars to add depth and movement.

Extends [``Effector``](../classes/Effector.md).

## Example

```ts
import { Chorus } from "@fluex/fluexgl-dsp";

const chorus = new Chorus({
    baseDelayMs: 15,
    depthMs: 8,
    rateHz: 1.5,
    mix: 0.5,
    feedback: 0.2
});

await chorus.initializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new Chorus(options: Partial<ChorusEffectOptions>): Chorus;
```

### Arguments

- ``options``: [``Partial<ChorusEffectOptions>``](../interfaces/ChorusEffectOptions.md)
  Chorus configuration options. Every field is optional and falls back to the class defaults below; numeric fields are also clamped (``baseDelayMs``/``depthMs``/``rateHz`` to ``>= 0``, ``mix``/``feedback`` to ``[0, 1]``).

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"Chorus"``.

### ``name: string``
Internal name for this effector. Defaults to ``"Chorus"``.

### ``baseDelayMs: number``
Base modulation delay, in milliseconds. Defaults to ``15``.

### ``depthMs: number``
Modulation depth, in milliseconds. Defaults to ``8``.

### ``rateHz: number``
Modulation rate, in Hertz. Defaults to ``1.5``.

### ``mix: number``
Dry/wet mix, between ``0`` and ``1``. Defaults to ``0.5``.

### ``feedback: number``
Feedback amount, between ``0`` and ``1``. Defaults to ``0.2``.

### ``strictMode: StrictMode``
Validation mode passed through to the processor. Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the chorus effect by creating the AudioWorklet processor node (``ChorusProcessor``) with the current option values.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the AudioWorklet node.

#### Returns

- ``Promise<void>``

- - -

### ``returnOptionsAsObject(): ChorusEffectOptions``

Returns a plain object snapshot of the current ``baseDelayMs``, ``depthMs``, ``rateHz``, ``mix``, ``feedback`` and ``strictMode`` values. Used internally to initialize the AudioWorklet node.

#### Returns

- [``ChorusEffectOptions``](../interfaces/ChorusEffectOptions.md)

- - -

### ``setBaseDelayMs(value: number): boolean``

Updates ``baseDelayMs`` (clamped to ``>= 0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean`` - ``true`` if the message was sent to the processor, ``false`` if the effect has not been initialized yet.

- - -

### ``setDepthMs(value: number): boolean``

Updates ``depthMs`` (clamped to ``>= 0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean``

- - -

### ``setRateHz(value: number): boolean``

Updates ``rateHz`` (clamped to ``>= 0``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean``

- - -

### ``setMix(value: number): boolean``

Updates ``mix`` (clamped to ``[0, 1]``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean``

- - -

### ``setFeedback(value: number): boolean``

Updates ``feedback`` (clamped to ``[0, 1]``) and forwards the change to the AudioWorklet processor.

#### Arguments

- ``value``: ``number``

#### Returns

- ``boolean``

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md) (``incoming-processor-message``, ``incoming-processor-warning``, ``incoming-processor-error``, ``processor-wasm-instantiated``).

## Getters and setters

This class does not define public getters or setters.
