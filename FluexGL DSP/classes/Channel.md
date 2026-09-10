# Class ``Channel``

An audio routing unit that can host an effect chain, attach audio clips via an internal [``AudioClipPlayer``](./AudioClipPlayer.md), and send its output to another [``Channel``](./Channel.md) or the [``Master``](./Master.md).

## Example

```ts
import { Channel } from "@fluex/fluexgl-dsp";

const channel = new Channel(context);
channel.send(master);
```

- - -

## Constructor
Constructs a new Channel and initializes its internal audio nodes (input → effects → panner → analyser → gain → output) and its [``AudioClipPlayer``](./AudioClipPlayer.md).

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
Stereo panner node for left/right balance control inside this channel.

### ``analyserNode: AnalyserNode | null``
Analyser node used for visualization / analysis of this channel's signal.

### ``gainNode: GainNode | null``
Gain node used to control the channel's volume after analysis.

### ``output: AudioNode | null``
Output node of this channel. This node is connected to other channels when calling ``send()``.

### ``effects: Effector[]``
List of attached [``Effector``](./Effector.md) instances. These are wired between ``input`` and ``stereoPannerNode``.

### ``context: AudioContext | null``
The AudioContext this channel was constructed with.

### ``sends: Channel[]``
Channels this channel is currently connected to via ``send()``.

### ``audioClipPlayer: AudioClipPlayer | null``
[``AudioClipPlayer``](./AudioClipPlayer.md) owned by this channel. Used to attach and play [``AudioClip``](./AudioClip.md) instances into this channel.

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
Publicly re-runs the internal effect chain rebuild (reconnects ``input`` through the active effects into ``stereoPannerNode``). Useful if the automatic rebuild did not run as expected.

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

### ``unsend(channel: Channel | Master): void``
Disconnects this channel from a previously linked [``Channel``](./Channel.md) or [``Master``](./Master.md) and removes it from ``sends``.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target that should stop receiving this channel's signal.

#### Returns
- ``void``

### ``unsendToAllChannels(): void``
Disconnects this channel from all channels currently stored in ``sends``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``hasAudioClipPlayer(): boolean``
Returns whether this channel has a constructed [``AudioClipPlayer``](./AudioClipPlayer.md).

#### Arguments
No arguments

#### Returns
- ``boolean`` - ``true`` if ``audioClipPlayer`` is defined, otherwise ``false``.

### ``attachAudioClip(audioClip: AudioClip): Channel``
Attaches an [``AudioClip``](./AudioClip.md) to this channel via its internal [``AudioClipPlayer``](./AudioClipPlayer.md). Throws if this channel has no ``audioClipPlayer``.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md) - The audio clip to attach.

#### Returns
- ``Channel`` - The same channel. Can be used to stack methods.

### ``volume(volume?: number): number``
Gets or sets this channel's gain. When ``volume`` is provided (and truthy), it is written to ``gainNode.gain`` at the current context time. Throws if the channel has no ``context`` or ``gainNode``.

#### Arguments
- ``volume?``: ``number`` - New gain value to apply. Omit to just read the current value.

#### Returns
- ``number`` - The value that was set, or the current ``gainNode.gain.value`` when no argument is given.

### ``pan(pan?: number): number``
Gets or sets this channel's stereo pan. When ``pan`` is provided (and truthy), it is written to ``stereoPannerNode.pan`` at the current context time. Throws if the channel has no ``context`` or ``stereoPannerNode``.

#### Arguments
- ``pan?``: ``number`` - New pan value to apply (between -1 and 1). Omit to just read the current value.

#### Returns
- ``number`` - The value that was set, or the current ``stereoPannerNode.pan.value`` when no argument is given.

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

This class does not define public getters or setters.

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

### Example 3: routing channels together
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
