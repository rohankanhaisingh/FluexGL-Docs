# Class ``MonoDelay``

A single-tap (mono) delay audio effector.

> **Placeholder class.** ``MonoDelay`` is currently declared as an empty class (``export class MonoDelay {}``) — it does not extend [``Effector``](../classes/Effector.md), and has no properties, methods, or constructor arguments of its own. It cannot be attached to a [``Channel``](../classes/Channel.md) or [``Master``](../classes/Master.md) via ``addEffect()``/``attachEffect()`` yet, since those expect an ``Effector`` instance. A ``MonoDelayProcessor`` AudioWorklet processor name is already reserved in ``AudioWorkletProcessorNames``, suggesting this class is planned but not yet wired up.
> <!-- TODO: verify -- confirm with the maintainer whether this is planned for a future release. -->

## Example

```ts
import { MonoDelay } from "@fluex/fluexgl-dsp";

const delay = new MonoDelay();
```

- - -

## Constructor

```ts
new MonoDelay(): MonoDelay;
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
