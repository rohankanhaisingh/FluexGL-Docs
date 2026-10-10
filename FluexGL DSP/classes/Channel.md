# Class ``Channel``

An audio routing unit that can host an effect chain, attach audio clips via an internal [``AudioClipPlayer``](./AudioClipPlayer.md), and send its output to another [``Channel``](./Channel.md) or the [``Master``](./Master.md).

A channel is kept as light as possible, because a game can have many of them: a new channel only has two audio nodes (``input`` and ``gainNode``). The stereo panner, the analyser and the clip player are only created when they are used.

In a game, use channels as **buses** (for example "Effects", "Entities", "UI", "Ambience" and "Voice chat"), not one per game object. Give a game object a [``SpatialAudioSource``](./SpatialAudioSource.md) instead, and route it to a bus with its ``bus`` option. See [Example 12: Game audio architecture](../examples/12-game-audio-architecture.md).

A channel can send to multiple targets (splitting the signal) and receive from multiple channels (merging the signal). To split a signal into its left/right or mid/side parts, see [``StereoMono.split()``](../effects/StereoMono.md). To receive audio from a microphone or a remote stream, use the subclass [``InputChannel``](./InputChannel.md).

## Example

```ts
import { Channel } from "@fluex/fluexgl-dsp";

const channel = new Channel(context);
channel.send(master);
```

- - -

## Constructor
Constructs a new Channel with two audio nodes: ``input`` and ``gainNode`` (which is also ``output``). The full chain is

```
input -> [effects] -> [stereo panner] -> [analyser] -> gain (output)
```

where the parts in brackets are only present when used: effects once added, the stereo panner once the channel is panned away from the centre (``pan()``), and the analyser once enabled (``enableAnalyser()``). The [``AudioClipPlayer``](./AudioClipPlayer.md) is created the first time ``audioClipPlayer`` is used.

```ts
new Channel(context: AudioContext, label?: string): Channel;
```

### Arguments
- ``context``: ``AudioContext`` - The AudioContext used to create this channel's internal audio nodes.
- ``label?``: ``string`` - Optional label for this channel. Defaults to ``"Channel"``.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing a new channel. Should NOT be changed.

### ``label: string``
A custom label for this channel. Can be changed.

### ``input: AudioNode | null``
Input node for this channel. Internally created as a GainNode and used as the start of the routing chain.

### ``stereoPannerNode: StereoPannerNode | null``
Stereo panner node for left/right balance control inside this channel. ``null`` until the channel is panned away from the centre with ``pan()``.

### ``analyserNode: AnalyserNode | null``
Analyser node used for visualization / analysis of this channel's signal. ``null`` until ``enableAnalyser()`` is called. Channels have no analyser by default, because it costs processing time on every channel.

### ``gainNode: GainNode | null``
Gain node used to control the channel's volume. The last node of the channel.

### ``output: AudioNode | null``
Output node of this channel. This node is connected to other channels when calling ``send()``. The same node as ``gainNode``.

### ``effects: Effector[]``
List of attached [``Effector``](./Effector.md) instances. These are wired between ``input`` and the stereo panner (or the analyser, or ``gainNode``, whichever comes first).

### ``context: AudioContext | null``
The AudioContext this channel was constructed with.

### ``sends: Channel[]``
Channels this channel is currently connected to via ``send()``.

### ``masters: Master[]``
[``Master``](./Master.md) channels this channel is attached to. Maintained by [``master.attachChannel()``](./Master.md) and [``master.detachChannel()``](./Master.md), so ``send(master)`` and ``unsend(master)`` keep it up to date. Should not be changed directly.

- - -

## Methods

### ``addEffect(effect: Effector): Channel``
Adds an [``Effector``](./Effector.md) to this channel, initializes it using this channel's ``AudioContext``, and rebuilds the internal effect chain routing. Throws if the channel has no ``AudioContext`` or if the effect was already added.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to add.

#### Returns
- ``Channel`` - The same channel. Can be used to stack methods.

### ``attachEffect(effect: Effector): Channel``
Alias for ``addEffect(effect)``.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to attach.

#### Returns
- ``Channel`` - The same channel. Can be used to stack methods.

### ``removeEffect(effect: Effector): void``
Removes an attached [``Effector``](./Effector.md) from this channel, disconnects its audio node, and rebuilds the internal effect chain routing. Logs an error if the effect is not part of this channel.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to remove.

#### Returns
- ``void``

### ``removeAllEffects(): void``
Removes every effect currently attached to this channel by calling ``removeEffect()`` for each one.

> Before version 0.5.2, every other effect was skipped. Effects are now all removed.

#### Arguments
No arguments

#### Returns
- ``void``

### ``detachEffect(effect: Effector): void``
Alias for ``removeEffect(effect)``.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to detach.

#### Returns
- ``void``

### ``detachAllEffects(): void``
Alias for ``removeAllEffects()``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``rebuildEffectChain(): void``
Publicly re-runs the internal effect chain rebuild (reconnects ``input`` through the active effects, the stereo panner and the analyser into ``gainNode``). Effects are connected through their [``inputNode`` and ``outputNode``](./Effector.md#getters-and-setters), so both AudioWorklet effects and native effects (such as [``Compressor``](../effects/Compressor.md)) can be mixed in one chain. Useful if the automatic rebuild did not run as expected.

#### Arguments
No arguments

#### Returns
- ``void``

### ``send(channel: Channel | Master): void``
Connects this channel's ``output`` to another [``Channel``](./Channel.md) (to its ``input``) or to the [``Master``](./Master.md). Prevents self-links, mismatched AudioContexts, duplicate links, and feedback loops.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target that should receive this channel's signal.

#### Returns
- ``void``

### ``unsend(channel: Channel | Master): boolean``
Disconnects this channel from a previously linked [``Channel``](./Channel.md) or [``Master``](./Master.md) and removes it from ``sends`` (or ``masters``).

Unsending from a target this channel is not sent to does nothing and does not log an error, so it is safe to call at any time, even when ``debugger.breakOnError`` is enabled.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target that should stop receiving this channel's signal.

#### Returns
- ``boolean`` - ``true`` when the link has been removed, ``false`` when there was no link.

### ``isSentTo(channel: Channel | Master): boolean``
Returns whether the signal of this channel is currently sent to the given [``Channel``](./Channel.md) or [``Master``](./Master.md). Useful for toggles.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md)

#### Returns
- ``boolean``

### ``unsendToAllChannels(): void``
Disconnects this channel from all channels currently stored in ``sends``. Master channels are not affected; use ``unsendFromAllMasters()`` or ``unsendFromAll()`` for those.

#### Arguments
No arguments

#### Returns
- ``void``

### ``unsendFromAllMasters(): void``
Detaches this channel from every [``Master``](./Master.md) channel it is attached to.

#### Arguments
No arguments

#### Returns
- ``void``

### ``unsendFromAll(): void``
Removes every outgoing link of this channel, both to channels and to master channels.

#### Arguments
No arguments

#### Returns
- ``void``

### ``hasAudioClipPlayer(): boolean``
Returns whether audio clips can be attached to this channel. The [``AudioClipPlayer``](./AudioClipPlayer.md) itself is created on demand, so this is ``true`` for every channel that has not been disposed.

#### Arguments
No arguments

#### Returns
- ``boolean``

### ``attachAudioClip(audioClip: AudioClip): Channel``
Attaches an [``AudioClip``](./AudioClip.md) to this channel via its internal [``AudioClipPlayer``](./AudioClipPlayer.md), creating the player if needed. Throws if the channel has been disposed.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md) - The audio clip to attach.

#### Returns
- ``Channel`` - The same channel. Can be used to stack methods.

### ``volume(volume?: number): number``
Gets or sets this channel's gain. When ``volume`` is provided (including ``0``), it is written to ``gainNode.gain`` at the current context time. Throws if the channel has no ``context`` or ``gainNode``.

#### Arguments
- ``volume?``: ``number`` - New gain value to apply. Omit to just read the current value.

#### Returns
- ``number`` - The value that was set, or the current ``gainNode.gain.value`` when no argument is given.

### ``pan(pan?: number): number``
Gets or sets this channel's stereo pan. The ``StereoPannerNode`` is created the first time the channel is panned away from the centre; ``pan(0)`` on a channel without a panner does nothing. Once created, the panner stays. Throws if the channel has no ``context``.

#### Arguments
- ``pan?``: ``number`` - New pan value to apply (between -1 and 1). Omit to just read the current value.

#### Returns
- ``number`` - The value that was set, or the current pan when no argument is given (``0`` when the channel has no panner).

### ``enableAnalyser(options?: AnalyserOptions): AnalyserNode``
Inserts an ``AnalyserNode`` after the effects (and the stereo panner) of this channel, and returns it. Calling it again returns the same node; ``options`` only apply when the node is created. Throws if the channel has no ``context``.

#### Arguments
- ``options?``: ``AnalyserOptions`` - Native Web Audio analyser options, such as ``fftSize`` and ``smoothingTimeConstant``.

#### Returns
- ``AnalyserNode`` - The channel's analyser, also available as ``analyserNode``.

### ``disableAnalyser(): void``
Removes the analyser created by ``enableAnalyser()``. Does nothing when the channel has no analyser.

#### Arguments
No arguments

#### Returns
- ``void``

### ``dispose(): void``
Stops the clips of this channel, removes all of its outgoing links (to channels and master channels) and releases its audio nodes. Channels that send to this channel are not changed; call ``unsend(channel)`` on them yourself. Until then their signal simply ends here.

#### Arguments
No arguments

#### Returns
- ``void``

### ``getEffectsByLabel(label: string): Effector[]``
Returns all attached effects whose ``label`` matches the given value.

#### Arguments
- ``label``: ``string`` - The label to match against.

#### Returns
- ``Effector[]``

### ``getFirstEffectByLabel(label: string): Effector | null``
Returns the first attached effect whose ``label`` matches the given value, or ``null`` if none match.

#### Arguments
- ``label``: ``string`` - The label to match against.

#### Returns
- ``Effector | null``

### ``getEffectById(id: string): Effector[]``
Returns all attached effects whose ``id`` matches the given value.

#### Arguments
- ``id``: ``string`` - The effect id to match against.

#### Returns
- ``Effector[]``

### ``getFirstEffectById(id: string): Effector | null``
Returns the first attached effect whose ``id`` matches the given value, or ``null`` if none match.

#### Arguments
- ``id``: ``string`` - The effect id to match against.

#### Returns
- ``Effector | null``

### ``moveEffectToIndex(effect: Effector, index: number | ArrayPosition): void``
Moves an already-attached effect to a new position in the ``effects`` chain and rebuilds the routing. Throws if the effect cannot be found, or if more than one effect shares the same id.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect to move. Must already be attached to this channel.
- ``index``: ``number | ArrayPosition`` - Either an absolute array index, or one of the named positions ``"start"``, ``"end"``, ``"one-after-start"``, ``"one-before-end"``. Out-of-range numeric indexes are clamped.

#### Returns
- ``void``

## Events

This class does not emit custom events.

- - -

## Getters and setters

### ``get audioClipPlayer(): AudioClipPlayer | null``
The [``AudioClipPlayer``](./AudioClipPlayer.md) of this channel, used to attach and play [``AudioClip``](./AudioClip.md) instances into it. Created the first time it is read, so channels that never play clips themselves do not carry an extra node. ``null`` after ``dispose()``.

- - -

## Examples

### Example 1: creating a channel using the Channel class.
```ts
import { DspPipeline, Channel } from "@fluex/fluexgl-dsp";

(async function () {

    const pipeline = new DspPipeline({
        pathToWasm: "/FluexGL-DSP-WASM/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/FluexGL-DSP-WASM/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const context = audioDevice.getContext();
    const master = audioDevice.getMasterChannel();

    const channel = new Channel(context);
    channel.send(master);
})()
```

### Example 2: creating channel without using the Channel class.
```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

(async function () {

    const pipeline = new DspPipeline({
        pathToWasm: "/FluexGL-DSP-WASM/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/FluexGL-DSP-WASM/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const context = audioDevice.getContext();
    const master = audioDevice.getMasterChannel();

    const channel = audioDevice.createChannel();
    channel.send(master);
})()
```

### Example 3: routing channels together (series)
```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

(async function () {

    const pipeline = new DspPipeline({
        pathToWasm: "/FluexGL-DSP-WASM/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/FluexGL-DSP-WASM/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const master = audioDevice.getMasterChannel();

    const channel1 = audioDevice.createChannel();
    const channel2 = audioDevice.createChannel();
    const channel3 = audioDevice.createChannel();

    channel1.send(channel2);
    channel2.send(channel3);
    channel3.send(master);
})()
```

### Example 4: toggling a send without errors
```ts
const master = audioDevice.getMasterChannel();
const channel = audioDevice.createChannel();

channel.send(master);

button.addEventListener("click", () => {
    if (channel.isSentTo(master)) {
        channel.unsend(master);
    } else {
        channel.send(master);
    }
});

// Safe: unsending something that is not linked just returns false.
channel.unsend(master);
channel.unsend(master);
```

### Example 5: splitting and merging
```ts
const source = audioDevice.createChannel("Source");
const dry = audioDevice.createChannel("Dry");
const wet = audioDevice.createChannel("Wet");
const bus = audioDevice.createChannel("Bus");

// Split: one channel sends to two channels.
source.send(dry);
source.send(wet);

wet.addEffect(new Reverb());

// Merge: two channels send to the same channel.
dry.send(bus);
wet.send(bus);

bus.send(audioDevice.getMasterChannel());
```

### Example 6: a meter on a bus
```ts
const effects = audioDevice.createChannel("Effects");
effects.send(audioDevice.getMasterChannel());

const analyser = effects.enableAnalyser({ fftSize: 1024 });
const samples = new Float32Array(analyser.fftSize);

function meter() {
    analyser.getFloatTimeDomainData(samples);
    const peak = samples.reduce((max, v) => Math.max(max, Math.abs(v)), 0);
    meterElement.style.width = `${peak * 100}%`;
    requestAnimationFrame(meter);
}

meter();

// No longer needed:
effects.disableAnalyser();
```
