# Class ``StereoPanner``

Positions a signal in the stereo field and controls its stereo width. Built from native Web Audio nodes, so it does not require WebAssembly.

Extends [``Effector``](../classes/Effector.md).

## Signal flow

```
input (forced to stereo) -> splitter -> width matrix -> merger -> StereoPannerNode -> output
```

The width matrix is a mid/side operation that scales the side signal (``L - R``) by ``width``:

```
out L = L * (1 + width) / 2 + R * (1 - width) / 2
out R = L * (1 - width) / 2 + R * (1 + width) / 2
```

| ``width`` | Result |
|---|---|
| ``0`` | Mono: both channels carry ``(L + R) / 2`` |
| ``1`` | Unchanged |
| ``2`` | Extra wide: the side signal is doubled |

A mono input is upmixed to both channels first, so ``width`` has no effect on it, but ``pan`` does.

## Example

```ts
import { StereoPanner } from "@fluex/fluexgl-dsp";

const panner = new StereoPanner({ pan: -0.3, width: 1.4 });

channel.addEffect(panner);

panner.setWidth(0); // collapse to mono
```

- - -

## Constructor

```ts
new StereoPanner(options?: Partial<StereoPannerOptions>): StereoPanner;
```

### Arguments

- ``options?``: [``Partial<StereoPannerOptions>``](../interfaces/StereoPannerOptions.md)

- - -

## Properties

### ``pan: number``
Position, between ``-1`` (left) and ``1`` (right). Defaults to ``0``.

### ``width: number``
Stereo width, between ``0`` (mono) and ``2`` (extra wide). Defaults to ``1``.

### ``inputGainNode: GainNode | null``
Input node, forced to 2 channels (``channelCountMode: "explicit"``). ``null`` until attached.

### ``pannerNode: StereoPannerNode | null``
Output node. ``null`` until attached.

``label`` and ``name`` default to ``"StereoPanner"``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the native nodes.

### ``returnOptionsAsObject(): StereoPannerOptions``

Returns ``{ pan, width }``.

### ``setPan(pan: number): boolean``

Sets the position, clamped between ``-1`` and ``1``, with a 10 ms ramp.

#### Arguments

- ``pan: number``

#### Returns

- ``boolean`` - ``true`` when applied to the audio node, ``false`` when the effect is not attached yet (the value is used on attachment).

### ``setWidth(width: number): boolean``

Sets the stereo width, clamped between ``0`` and ``2``, with a 10 ms ramp.

#### Arguments

- ``width: number``

#### Returns

- ``boolean`` - ``true`` when applied to the audio nodes, ``false`` when the effect is not attached yet.

## Getters and setters

### ``get inputNode(): AudioNode | null``
Returns ``inputGainNode``.

### ``get outputNode(): AudioNode | null``
Returns ``pannerNode``.

- - -

## Events

``StereoPanner`` does not use an AudioWorklet processor, so it does not dispatch processor events.

## Notes

Every [``Channel``](../classes/Channel.md) already has a ``StereoPannerNode`` (``channel.pan()``). Use this effect when you also need width control, or panning at a specific position in an effect chain.
