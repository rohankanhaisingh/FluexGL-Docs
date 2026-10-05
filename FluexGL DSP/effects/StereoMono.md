# Class ``StereoMono``

Routes, mixes and delays the left and right channel of a signal: stereo, mono, swapped, one side only, mid or side. Together with ``StereoMono.split()`` it is used to split a signal into two branches, process each branch differently, and merge them again, for example to create a pseudo surround effect. Built from native Web Audio nodes, so it does not require WebAssembly.

Extends [``Effector``](../classes/Effector.md).

## Signal flow

```
input (forced to stereo) -> splitter -> 2x2 matrix -> delay per side -> merger -> output
```

The matrix calculates both outputs from both inputs:

```
out L = ll * L + rl * R
out R = lr * L + rr * R
```

With ``M = (L + R) / 2`` (mid) and ``S = (L - R) / 2`` (side), the modes are:

| ``mode`` | Output (L, R) | Use |
|---|---|---|
| ``"stereo"`` | ``(L, R)`` | Unchanged. Default. |
| ``"mono"`` | ``(M, M)`` | Mono compatibility check, mono sources. |
| ``"mid"`` | ``(M, M)`` | Same as ``"mono"``; the counterpart of ``"side"``. |
| ``"side"`` | ``(S, -S)`` | Only the stereo difference: what makes a signal sound wide. |
| ``"swap"`` | ``(R, L)`` | Swaps the sides. |
| ``"left"`` | ``(L, 0)`` | Keeps the left side in place, silences the right side. |
| ``"right"`` | ``(0, R)`` | Keeps the right side in place, silences the left side. |
| ``"left-to-both"`` | ``(L, L)`` | The left side on both speakers. |
| ``"right-to-both"`` | ``(R, R)`` | The right side on both speakers. |

The modes are designed so a split can be merged back losslessly: ``"left"`` + ``"right"`` = ``(L, R)``, and ``"mid"`` + ``"side"`` = ``(M + S, M - S)`` = ``(L, R)``.

After the matrix, each side has its own ``DelayNode`` (0 to 100 ms) and an optional polarity inversion. A short delay on one side (1 to 30 ms, the Haas effect) widens the image; an inverted polarity on one side makes a signal sound diffuse. Both are the building blocks of pseudo surround.

A mono input (such as a microphone) is upmixed to both channels first. Its side signal is then zero, so ``"side"`` is silent until the left and right side differ, for example after a delay on one side.

All parameter changes use a 10 ms ramp, so switching modes does not click.

## Example

```ts
import { StereoMono } from "@fluex/fluexgl-dsp";

const stereoMono = new StereoMono({ mode: "mono" });

channel.addEffect(stereoMono);

stereoMono.setMode("stereo");
stereoMono.setDelayRight(12); // Haas widening
```

- - -

## Constructor

```ts
new StereoMono(options?: Partial<StereoMonoOptions>): StereoMono;
```

### Arguments

- ``options?``: [``Partial<StereoMonoOptions>``](../interfaces/StereoMonoOptions.md)

- - -

## Properties

### ``mode: StereoMonoMode``
How the left and right channel are routed. Defaults to ``"stereo"``. See [``StereoMonoMode``](../interfaces/StereoMonoMode.md).

### ``delayLeftMs: number``
Delay of the left output (ms), between ``0`` and ``100``. Defaults to ``0``.

### ``delayRightMs: number``
Delay of the right output (ms), between ``0`` and ``100``. Defaults to ``0``.

### ``invertLeft: boolean``
Whether the polarity of the left output is inverted. Defaults to ``false``.

### ``invertRight: boolean``
Whether the polarity of the right output is inverted. Defaults to ``false``.

### ``inputGainNode: GainNode | null``
Input node, forced to 2 channels (``channelCountMode: "explicit"``). ``null`` until attached.

### ``mergerNode: ChannelMergerNode | null``
Output node. ``null`` until attached.

``label`` and ``name`` default to ``"StereoMono"``.

- - -

## Static methods

### ``StereoMono.split(source: Channel, mode?: StereoSplitMode): [Channel, Channel]``

Splits the signal of a channel into two new [``Channel``](../classes/Channel.md)s on the same ``AudioContext``. Each branch gets a ``StereoMono`` effect as its first effect, and ``source`` is sent to both branches. Merge the branches by sending both to the same channel.

| ``mode`` | First branch | Second branch | Labels |
|---|---|---|---|
| ``"left-right"`` (default) | ``(L, 0)`` | ``(0, R)`` | ``"<label> L"``, ``"<label> R"`` |
| ``"mid-side"`` | ``(M, M)`` | ``(S, -S)`` | ``"<label> Mid"``, ``"<label> Side"`` |

The source channel keeps its other sends. Detach it from the master channel (``source.unsend(master)``) if only the branches should be heard, otherwise the original signal is heard as well.

The ``StereoMono`` effect of a branch is available as ``branch.effects[0]``, so you can add a delay or polarity inversion to it directly.

#### Arguments

- ``source``: [``Channel``](../classes/Channel.md) - The channel to split. Throws when it has no ``AudioContext``.
- ``mode?``: [``StereoSplitMode``](../interfaces/StereoSplitMode.md) - Defaults to ``"left-right"``.

#### Returns

- ``[Channel, Channel]`` - The two branches.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the native nodes.

### ``returnOptionsAsObject(): StereoMonoOptions``

Returns ``{ mode, delayLeftMs, delayRightMs, invertLeft, invertRight }``.

### ``setMode(mode: StereoMonoMode): boolean``

Sets how the left and right channel are routed, with a 10 ms ramp.

#### Arguments

- ``mode``: [``StereoMonoMode``](../interfaces/StereoMonoMode.md)

#### Returns

- ``boolean`` - ``true`` when applied to the audio nodes, ``false`` when the mode is unknown or the effect is not attached yet (a valid mode is then used on attachment).

### ``setInvert(invertLeft: boolean, invertRight?: boolean): boolean``

Inverts the polarity of the left and/or right output. When ``invertRight`` is omitted, the current value is kept.

#### Returns

- ``boolean`` - ``true`` when applied to the audio nodes, ``false`` when the effect is not attached yet.

### ``setDelayLeft(ms: number): boolean``

Sets the delay of the left output, clamped between ``0`` and ``100`` ms, with a 10 ms ramp.

#### Returns

- ``boolean`` - ``true`` when applied to the audio node, ``false`` when the effect is not attached yet.

### ``setDelayRight(ms: number): boolean``

Sets the delay of the right output, clamped between ``0`` and ``100`` ms, with a 10 ms ramp.

#### Returns

- ``boolean`` - ``true`` when applied to the audio node, ``false`` when the effect is not attached yet.

## Getters and setters

### ``get inputNode(): AudioNode | null``
Returns ``inputGainNode``.

### ``get outputNode(): AudioNode | null``
Returns ``mergerNode``.

- - -

## Events

``StereoMono`` does not use an AudioWorklet processor, so it does not dispatch processor events.

- - -

## Examples

### Example 1: a mono button
```ts
const stereoMono = new StereoMono();
master.attachEffect(stereoMono);

monoButton.addEventListener("click", () => {
    stereoMono.setMode(stereoMono.mode === "mono" ? "stereo" : "mono");
});
```

### Example 2: processing the left and right side separately
```ts
const [left, right] = StereoMono.split(source, "left-right");
const merged = audioDevice.createChannel("Merged");

source.unsend(master);

right.addEffect(new LowPassFilter({ cutoff: 2000 }));

left.send(merged);
right.send(merged);
merged.send(master);
```

### Example 3: pseudo surround
```ts
const [mid, side] = StereoMono.split(source, "mid-side");
const surround = audioDevice.createChannel("Surround");

source.unsend(master);

// Turn the side signal into a diffuse "rear": delayed, darker and louder.
(side.effects[0] as StereoMono).setDelayRight(18);
side.addEffect(new LowPassFilter({ cutoff: 7000 }));
side.volume(1.4);

mid.send(surround);
side.send(surround);
surround.send(master);
```

See [Example 10: Splitting and merging](../examples/10-splitting-and-merging.md) for a complete walkthrough.

## Notes

- Why native nodes and not WebAssembly: routing, mixing and delaying are linear operations. Native Web Audio nodes run them on the audio thread without the block copies and JavaScript/WebAssembly calls of an AudioWorklet, and their parameters can be automated without clicks.
- [``StereoPanner``](./StereoPanner.md) controls the stereo **width** (``width: 0`` is mono as well). Use ``StereoMono`` when you need routing (swap, one side, mid/side) or a split.
