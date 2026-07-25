# Class ``Chorus``

A chorus audio effector that creates a thickened, widened sound by duplicating the signal,
modulating delay times, and mixing the result back with the dry signal.

This effector is typically used for pads, leads, and guitars to add depth and movement.

## Example

```ts
import { Chorus } from "@fluex/fluexgl-dsp";

const chorus = new Chorus({
    rate: 1.5,
    depth: 0.7,
    delayTime: 0.03,
    feedback: 0.2,
    mix: 0.5
});

await chorus.InitializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new Chorus(options?: Partial<ChorusOptions>): Chorus;
```

### Arguments

- ``options?``: ``Partial<ChorusOptions>``  
  Optional chorus configuration options.

- - -

## Properties

### ``label: string | null``
Custom label for this effector. Defaults to ``"Chorus"``.

### ``name: string``
Internal name for this effector. Defaults to ``"Chorus"``.

### ``inputNode: AudioNode | null``
The input node of the chorus effect.

### ``outputNode: AudioNode | null``
The output node of the chorus effect.

- - -

## Methods

### ``InitializeOnAttachment(context: AudioContext): Promise<void>``

Initializes the chorus effect by constructing internal delay, gain, and modulation nodes
and wiring them into the DSP graph.

#### Arguments

- ``context``: ``AudioContext``  
  The audio context used to construct the internal nodes.

#### Returns

- ``Promise<void>``

- - -

### ``SetOptions(options: Partial<ChorusOptions>): void``

Applies chorus parameters such as rate, depth, delay time, feedback, and mix.
Live-updates internal nodes when possible.

#### Arguments

- ``options``: ``Partial<ChorusOptions>``  
  Options to apply.

#### Returns

- ``void``
