# Class ``Effector``

Abstract base class for every DSP effect (e.g. [``Chorus``](../effects/Chorus.md), [``LowPassFilter``](../effects/LowPassFilter.md), [``HardClip``](../effects/HardClip.md)). Provides the common ``id``/``label``/``name`` bookkeeping, the ``audioWorkletNode`` slot, the ``inputNode``/``outputNode`` getters used to wire effects into a chain, and the event system used to receive messages coming back from the AudioWorklet processor.

Most effects run on an AudioWorklet (``audioWorkletNode``). Effects built from native Web Audio nodes, such as [``Compressor``](../effects/Compressor.md) and [``Limiter``](../effects/Limiter.md), override ``inputNode`` and ``outputNode`` instead.

``Effector`` cannot be instantiated directly — it is meant to be extended, with subclasses implementing ``initializeOnAttachment()``.

## Example

```ts
import { Effector } from "@fluex/fluexgl-dsp";

// Effector is abstract - you use one of its subclasses instead:
// const effect = new Chorus({ mix: 0.4 });
// channel.addEffect(effect);

effect.addEventListener("incoming-processor-error", (message) => {
    console.error(message);
});
```

- - -

## Constructor
``Effector`` has no public constructor of its own beyond the implicit one inherited by subclasses (it is declared ``abstract`` and is always constructed through a concrete subclass such as ``Chorus`` or ``LowPassFilter``).

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the effect. Should NOT be changed.

### ``label: string | null``
A custom label for this effect. Subclasses typically default this to their own name (e.g. ``"Chorus"``).

### ``name: string``
Internal name of the effect. Defaults to ``"Effector"`` and is overridden by subclasses.

### ``audioWorkletNode: AudioWorkletNode | null``
The underlying ``AudioWorkletNode`` (or, for effects that wrap a native node such as [``Analyser``](../effects/Analyser.md), a node cast to this type) created during ``initializeOnAttachment()``. ``null`` until the effect has been attached to a [``Channel``](./Channel.md) or [``Master``](./Master.md).

### ``context: AudioContext | null``
The ``AudioContext`` this effect was initialized with. ``null`` until attached.

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``
Abstract method every subclass must implement. Called automatically when the effect is attached to a [``Channel``](./Channel.md) or [``Master``](./Master.md) (via ``addEffect``/``attachEffect``); responsible for constructing the effect's audio nodes (its ``audioWorkletNode``, or the native nodes returned by ``inputNode``/``outputNode``). The nodes must be created synchronously, before the first ``await``, because the chain is rebuilt right after this method is called.

#### Arguments
- ``context``: ``AudioContext`` - The audio context to initialize the effect with.

#### Returns
- ``Promise<void>``

### ``addEventListener<K extends keyof EffectorEventMap>(event: K, cb: EffectorEventMap[K]): () => void``
Registers a listener for processor events.

#### Arguments
- ``event: K (keyof EffectorEventMap)`` - Event name (e.g. ``"incoming-processor-message"``).
- ``cb: (message: IncomingProcessorMessage) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``once<K extends keyof EffectorEventMap>(event: K, cb: EffectorEventMap[K]): () => void``
Registers a one-time event listener that automatically removes itself after the first call.

#### Arguments
- ``event: K (keyof EffectorEventMap)`` - Event name.
- ``cb: (message: IncomingProcessorMessage) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``removeEventListener<K extends keyof EffectorEventMap>(event: K, cb: EffectorEventMap[K]): Effector``
Removes a specific listener from an event.

#### Arguments
- ``event: K (keyof EffectorEventMap)`` - Event name.
- ``cb: (message: IncomingProcessorMessage) => void`` - Callback function.

#### Returns
- ``Effector`` - The same instance, for chaining.

### ``clearEventListeners(event?: keyof EffectorEventMap): Effector``
Clears event listeners.

#### Arguments
- ``event?: keyof EffectorEventMap`` - If provided, clears listeners only for that event.

#### Returns
- ``Effector`` - The same instance, for chaining.

## Events

All events carry an [``IncomingProcessorMessage``](../interfaces/IncomingProcessorMessage.md) payload, dispatched from messages posted back by the AudioWorklet processor.

### ``"incoming-processor-message"``
A generic message was received from the processor.

### ``"incoming-processor-warning"``
<!-- TODO: verify -- in the current source, the internal message handler routes processor messages of type "warning" into the "incoming-processor-message" listeners instead of "incoming-processor-warning" (likely a bug). Documented here as declared in EffectorEventMap; the mismatch is called out so it isn't mistaken for intended behavior. -->
A warning message was received from the processor.

### ``"incoming-processor-error"``
An error message was received from the processor.

### ``"processor-wasm-instantiated"``
Fired once the processor reports that its WebAssembly module has been instantiated.

### ``"initialized-on-channel-attachment"``
Declared on [``EffectorEventMap``](../interfaces/EffectorEventMap.md) but not currently dispatched anywhere in ``Effector`` itself.

- - -

## Getters and setters

### ``get inputNode(): AudioNode | null``
The node that receives the signal of this effect. [``Channel``](./Channel.md) and [``Master``](./Master.md) connect the previous node in the chain to it. Returns ``audioWorkletNode`` by default. ``null`` until the effect is attached.

### ``get outputNode(): AudioNode | null``
The node that outputs the processed signal of this effect. [``Channel``](./Channel.md) and [``Master``](./Master.md) connect it to the next node in the chain, and disconnect it when the chain is rebuilt or the effect is removed. Returns ``audioWorkletNode`` by default. ``null`` until the effect is attached.

Effects that consist of multiple native nodes override both getters. Only ``outputNode`` is disconnected during a rebuild, so connections inside the effect stay intact.

- - -

## Examples

### Example: listening for processor messages on any effect
```ts
import { Chorus } from "@fluex/fluexgl-dsp";

const chorus = new Chorus({ mix: 0.4 });

const unsubscribe = chorus.addEventListener("incoming-processor-message", (message) => {
    console.log(message.message);
});

channel.addEffect(chorus);

// Later
unsubscribe();
```

### Example: a custom effect built from native nodes
```ts
import { Effector } from "@fluex/fluexgl-dsp";

class Telephone extends Effector {

    public name: string = "Telephone";
    public label: string | null = "Telephone";

    private highpass: BiquadFilterNode | null = null;
    private lowpass: BiquadFilterNode | null = null;

    public async initializeOnAttachment(context: AudioContext): Promise<void> {
        this.context = context;

        // Create the nodes synchronously: the chain is rebuilt right after this call.
        this.highpass = new BiquadFilterNode(context, { type: "highpass", frequency: 300 });
        this.lowpass = new BiquadFilterNode(context, { type: "lowpass", frequency: 3400 });
        this.highpass.connect(this.lowpass);
    }

    public get inputNode(): AudioNode | null {
        return this.highpass;
    }

    public get outputNode(): AudioNode | null {
        return this.lowpass;
    }
}

channel.addEffect(new Telephone());
```
