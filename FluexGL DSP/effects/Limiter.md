# Class ``Limiter``

A limiter audio effector.

> **Placeholder class.** ``Limiter`` is currently declared as an empty class (``export class Limiter {}``) — it does not extend [``Effector``](../classes/Effector.md), and has no properties, methods, or constructor arguments of its own. It cannot be attached to a [``Channel``](../classes/Channel.md) or [``Master``](../classes/Master.md) via ``addEffect()``/``attachEffect()`` yet, since those expect an ``Effector`` instance.
> <!-- TODO: verify -- confirm with the maintainer whether this is planned for a future release. -->

## Example

```ts
import { Limiter } from "@fluex/fluexgl-dsp";

const limiter = new Limiter();
```

- - -

## Constructor

```ts
new Limiter(): Limiter;
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
