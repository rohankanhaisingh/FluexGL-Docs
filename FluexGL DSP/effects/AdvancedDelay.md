# Class ``AdvancedDelay``

A stereo delay with every control of the delay engine. Use it for tape-style echoes, dub delays, or anything between a stereo delay and a ping-pong delay.

Extends [``Effector``](../classes/Effector.md) (through the internal ``DelayEngine`` base class).

All four delays (``MonoDelay``, ``StereoDelay``, ``PingPongDelay`` and [``AdvancedDelay``](./AdvancedDelay.md)) run the same WebAssembly delay engine in a different mode. Delay time changes glide smoothly over about 50 ms, so changing the time while audio plays gives a short pitch bend instead of a click. The delay keeps processing when its input goes silent, so echoes ring out naturally.

## Signal flow

```
            +----------------------------- dry ----------------------------+
input L/R --+-> delay line L/R --+--------------------------- wet ---------+-> mix -> output
                  ^              |
                  |              v
                  +-- drive <- high cut <- low cut <- cross feedback mix <-+
```

- **Cross feedback** blends the feedback of each side into the other: ``0`` keeps the sides separate (stereo delay), ``1`` swaps them on every repeat (ping-pong).
- **Low cut** and **high cut** filter the feedback path, so every repeat gets thinner and darker, like an analog or tape delay.
- **Drive** soft-saturates the feedback path. It warms up the repeats and keeps high feedback settings from running away.
- **Modulation** varies the delay time with a sine LFO (the right side runs 90 degrees ahead). Small depths (1 to 3 ms) give tape wow; larger depths give a chorus-like wobble.

## Example

```ts
import { AdvancedDelay } from "@fluex/fluexgl-dsp";

// A dark, warm tape echo.
const tape = new AdvancedDelay({
    delayLeftMs: 350,
    delayRightMs: 350,
    feedback: 0.6,
    crossFeedback: 0,
    lowCut: 150,
    highCut: 3500,
    modulationRate: 0.8,
    modulationDepth: 2,
    drive: 0.5,
    mix: 0.3
});

channel.addEffect(tape);

// Automate the time for a pitch-bending "tape stop"-like effect.
tape.setDelayLeftMs(700);
tape.setDelayRightMs(700);
```

- - -

## Constructor

```ts
new AdvancedDelay(options?: Partial<AdvancedDelayOptions>): AdvancedDelay;
```

### Arguments

- ``options?``: [``Partial<AdvancedDelayOptions>``](../interfaces/AdvancedDelayOptions.md)
  Every field is optional, falls back to the defaults below, and is clamped to its valid range.

- - -

## Properties

| Property | Range | Default |
|---|---|---|
| ``delayLeftMs: number`` | ``1`` to ``4000`` ms | ``375`` |
| ``delayRightMs: number`` | ``1`` to ``4000`` ms | ``500`` |
| ``feedback: number`` | ``0`` to ``0.98`` | ``0.45`` |
| ``crossFeedback: number`` | ``0`` (stereo) to ``1`` (ping-pong) | ``0.2`` |
| ``mix: number`` | ``0`` to ``1`` | ``0.35`` |
| ``lowCut: number`` | ``0`` (off) to ``2000`` Hz | ``120`` |
| ``highCut: number`` | ``0`` (off) or a frequency in Hz; at or above Nyquist is off | ``6000`` |
| ``modulationRate: number`` | ``0`` to ``10`` Hz | ``0.5`` |
| ``modulationDepth: number`` | ``0`` (off) to ``20`` ms | ``0`` |
| ``drive: number`` | ``0`` (off) to ``1`` | ``0.2`` |
| ``strictMode: StrictMode`` | | ``StrictMode.Disabled`` |

``label`` and ``name`` default to ``"AdvancedDelay"``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the ``AdvancedDelayProcessor`` AudioWorklet node with the current values.

### ``returnOptionsAsObject(): AdvancedDelayOptions``

Returns a snapshot of the current values.

### Setters

```ts
setDelayLeftMs(delayMs: number): boolean
setDelayRightMs(delayMs: number): boolean
setFeedback(feedback: number): boolean
setCrossFeedback(crossFeedback: number): boolean
setMix(mix: number): boolean
setLowCut(lowCut: number): boolean
setHighCut(highCut: number): boolean
setModulationRate(modulationRate: number): boolean
setModulationDepth(modulationDepth: number): boolean
setDrive(drive: number): boolean
```

Every setter clamps the value to the range in the table above, stores it, and sends it to the processor.

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor, ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md). The ``AdvancedDelayProcessor`` processor sends ``processor-wasm-instantiated`` when its WebAssembly module is ready, and an ``incoming-processor-message`` for every parameter change.

## Requirements

Runs on an AudioWorklet backed by WebAssembly. Attach it only after ``await pipeline.initializeDpsPipeline()``, on the ``AudioDevice`` returned by ``resolveDefaultAudioOutputDevice()``. Requires a worklet and WASM build from FluexGL-DSP-WebAssembly 0.4.9 or newer, which registers ``AdvancedDelayProcessor``.
