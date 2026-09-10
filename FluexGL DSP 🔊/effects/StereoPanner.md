# Class ``StereoPanner``

A stereo panner audio effector.

> **Placeholder class.** ``StereoPanner`` currently only declares a ``name`` property — it does not extend [``Effector``](../classes/Effector.md), and has no panning behavior, other properties, or methods implemented yet. It cannot be attached to a [``Channel``](../classes/Channel.md) or [``Master``](../classes/Master.md) via ``addEffect()``/``attachEffect()`` yet, since those expect an ``Effector`` instance. (Per-channel stereo panning is already available today through [``Channel.pan()``](../classes/Channel.md#panpan-number-number) / [``AudioClip.setPanLevel()``](../classes/AudioClip.md#setpanlevelpanlevel-number-audioclip).)
> <!-- TODO: verify -- confirm with the maintainer whether this is planned for a future release. -->

## Example

```ts
import { StereoPanner } from "@fluex/fluexgl-dsp";

const panner = new StereoPanner();
console.log(panner.name); // "StereoPanner"
```

- - -

## Constructor

```ts
new StereoPanner(): StereoPanner;
```

Takes no arguments. Uses the default (implicit) constructor.

- - -

## Properties

### ``name: string``
Internal name for this effector. Defaults to ``"StereoPanner"``.

- - -

## Methods

This class has no public methods.

- - -

## Events

This class does not emit any events.

## Getters and setters

This class does not define public getters or setters.
