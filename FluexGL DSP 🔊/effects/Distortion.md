# Class ``Distortion``

A distortion audio effector.

Extends [``Effector``](../classes/Effector.md).

> **Work in progress.** This class currently only declares its identity (``name``) and an empty ``initializeOnAttachment()`` implementation — it does not yet create an ``audioWorkletNode``, expose any tunable properties, or apply any actual distortion processing. Attaching it to a [``Channel``](../classes/Channel.md) or [``Master``](../classes/Master.md) will not audibly change the signal yet.
> <!-- TODO: verify -- confirm with the maintainer whether Distortion is mid-implementation or intentionally a stub pending a future release. -->

## Example

```ts
import { Distortion } from "@fluex/fluexgl-dsp";

const distortion = new Distortion();

await distortion.initializeOnAttachment(audioContext);
```

- - -

## Constructor

```ts
new Distortion(): Distortion;
```

Takes no arguments.

- - -

## Properties

### ``name: string``
Internal name for this effector. Defaults to ``"Distortion"``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Currently an empty implementation — no ``audioWorkletNode`` or ``context`` is set up.

#### Arguments

- ``context``: ``AudioContext``

#### Returns

- ``Promise<void>``

- - -

## Events

This class does not currently emit any events.

## Getters and setters

This class does not define public getters or setters.
