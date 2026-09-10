# Class ``Compressor``

A dynamic range compressor audio effector.

> **Placeholder class.** ``Compressor`` is currently declared as an empty class (``export class Compressor {}``) — it does not extend [``Effector``](../classes/Effector.md), and has no properties, methods, or constructor arguments of its own. It cannot be attached to a [``Channel``](../classes/Channel.md) or [``Master``](../classes/Master.md) via ``addEffect()``/``attachEffect()`` yet, since those expect an ``Effector`` instance. A ``CompressorProcessor`` AudioWorklet processor name is already reserved in ``AudioWorkletProcessorNames``, suggesting this class is planned but not yet wired up.
> <!-- TODO: verify -- confirm with the maintainer whether this is planned for a future release. -->

## Example

```ts
import { Compressor } from "@fluex/fluexgl-dsp";

const compressor = new Compressor();
```

- - -

## Constructor

```ts
new Compressor(): Compressor;
```

Takes no arguments. Uses the default (implicit) constructor.

- - -

## Properties

This class has no public properties.

- - -

## Methods

This class has no public methods.

- - -

## Events

This class does not emit any events.

## Getters and setters

This class does not define public getters or setters.
