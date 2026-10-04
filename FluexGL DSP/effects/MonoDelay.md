# Class ``MonoDelay``

A mono delay: the input is summed to mono and fed through a single delay line, so the echoes are identical on both channels.

Extends [``Effector``](../classes/Effector.md) (through the internal ``DelayEngine`` base class).

All four delays (``MonoDelay``, ``StereoDelay``, ``PingPongDelay`` and [``AdvancedDelay``](./AdvancedDelay.md)) run the same WebAssembly delay engine in a different mode. Delay time changes glide smoothly over about 50 ms, so changing the time while audio plays gives a short pitch bend instead of a click. The delay keeps processing when its input goes silent, so echoes ring out naturally.

## Example

```ts
import { MonoDelay } from "@fluex/fluexgl-dsp";

const delay = new MonoDelay({ delayMs: 250, feedback: 0.4, mix: 0.3 });

channel.addEffect(delay);

delay.setDelayMs(375);
```

- - -

## Constructor

```ts
new MonoDelay(options?: Partial<MonoDelayOptions>): MonoDelay;
```

### Arguments

- ``options?``: [``Partial<MonoDelayOptions>``](../interfaces/MonoDelayOptions.md)
  Every field is optional, falls back to the defaults below, and is clamped to its valid range.

- - -

## Properties

### ``label: string | null``
Defaults to ``"MonoDelay"``.

### ``name: string``
Defaults to ``"MonoDelay"``.

### ``delayMs: number``
Delay time in milliseconds, between ``1`` and ``4000``. Defaults to ``300``.

### ``feedback: number``
Amount of each echo that is fed back, between ``0`` and ``0.98``. Defaults to ``0.35``.

### ``mix: number``
Dry/wet mix, between ``0`` (dry) and ``1`` (wet only). Defaults to ``0.35``.

### ``strictMode: StrictMode``
Defaults to ``StrictMode.Disabled``.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``

Creates the ``MonoDelayProcessor`` AudioWorklet node with the current values. Called automatically by ``addEffect``/``attachEffect``.

### ``returnOptionsAsObject(): MonoDelayOptions``

Returns a snapshot of the current values.

### ``setDelayMs(delayMs: number): boolean``

Sets the delay time (ms), clamped between ``1`` and ``4000``. The time glides to the new value.

#### Arguments

- ``delayMs: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setFeedback(feedback: number): boolean``

Sets the feedback, clamped between ``0`` and ``0.98``.

#### Arguments

- ``feedback: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

### ``setMix(mix: number): boolean``

Sets the dry/wet mix, clamped between ``0`` and ``1``.

#### Arguments

- ``mix: number``

#### Returns

- ``boolean`` - ``true`` when the value was sent to the processor (or applied to the audio node), ``false`` when the effect is not attached yet. The value is stored either way and used on attachment.

- - -

## Events

Inherited from [``Effector``](../classes/Effector.md). The ``MonoDelayProcessor`` processor sends ``processor-wasm-instantiated`` when its WebAssembly module is ready, and an ``incoming-processor-message`` for every parameter change.

## Requirements

Runs on an AudioWorklet backed by WebAssembly. Attach it only after ``await pipeline.initializeDpsPipeline()``, on the ``AudioDevice`` returned by ``resolveDefaultAudioOutputDevice()``. Requires a worklet and WASM build from FluexGL-DSP-WebAssembly 0.4.9 or newer, which registers ``MonoDelayProcessor``.
