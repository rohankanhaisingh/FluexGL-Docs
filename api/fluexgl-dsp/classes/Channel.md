# Class ``Channel``

An audio routing unit that can host an effect chain, attach audio clips via an internal [``AudioClipPlayer``](./AudioClipPlayer.md), and send its output to another [``Channel``](./Channel.md) or the [``Master``](./Master.md).

## Example

```ts
import { Channel } from "@fluex/fluexgl-dsp";

const channel = new Channel(context);
channel.Send(master);
```

- - -

## Constructor
Constructs a new Channel and initializes its internal audio nodes (input → effects → panner → analyser → gain → output) and its [``AudioClipPlayer``](./AudioClipPlayer.md).

```ts
new Channel(context: AudioContext): Channel;
```

### Arguments
- ``context``: ``AudioContext`` - The AudioContext used to create this channel's internal audio nodes.

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
Output node of this channel. This node is connected to other channels when calling ``Send()``.

### ``effects: Effector[]``
List of attached [``Effector``](./Effector.md) instances. These are wired between ``input`` and ``stereoPannerNode``.

### ``context: AudioContext | null``
The AudioContext this channel was constructed with.

### ``sends: Channel[]``
Channels this channel is currently connected to via ``Send()``.

### ``audioClipPlayer: AudioClipPlayer | null``
[``AudioClipPlayer``](./AudioClipPlayer.md) owned by this channel. Used to attach and play [``AudioClip``](./AudioClip.md) instances into this channel.

- - -

## Methods

### ``AddEffect(effect: Effector): void``
Adds an [``Effector``](./Effector.md) to this channel, initializes it using this channel's ``AudioContext``, and rebuilds the internal effect chain routing.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to add.

#### Returns
- ``void``

### ``AttachEffect(effect: Effector): void``
Alias for ``AddEffect(effect)``.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to attach.

#### Returns
- ``void``

### ``RemoveEffect(effect: Effector): void``
Removes an attached [``Effector``](./Effector.md) from this channel, disconnects its audio node, and rebuilds the internal effect chain routing.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to remove.

#### Returns
- ``void``

### ``DetachEffect(effect: Effector): void``
Alias for ``RemoveEffect(effect)``.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to detach.

#### Returns
- ``void``

### ``Send(channel: Channel | Master): void``
Connects this channel's ``output`` to another [``Channel``](./Channel.md) (to its ``input``) or to the [``Master``](./Master.md). Prevents self-links, mismatched AudioContexts, duplicate links, and feedback loops.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target that should receive this channel's signal.

#### Returns
- ``void``

### ``Unsend(channel: Channel | Master): void``
Disconnects this channel from a previously linked [``Channel``](./Channel.md) or [``Master``](./Master.md) and removes it from ``sends``.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target that should stop receiving this channel's signal.

#### Returns
- ``void``

### ``HasAudioClipPlayer(): boolean``
Returns whether this channel has a constructed [``AudioClipPlayer``](./AudioClipPlayer.md).

#### Arguments
No arguments

#### Returns
- ``boolean`` - ``true`` if ``audioClipPlayer`` is defined, otherwise ``false``.

### ``UnsendToAllChannels(): void``
Disconnects this channel from all channels currently stored in ``sends``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``AttachAudioClip(audioClip: AudioClip): void``
Attaches an [``AudioClip``](./AudioClip.md) to this channel via its internal [``AudioClipPlayer``](./AudioClipPlayer.md).

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md) - The audio clip to attach.

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

    await pipeline.InitializeDpsPipeline();

    const audioDevice = await pipeline.ResolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const context = audioDevice.GetContext();
    const master = audioDevice.GetMasterChannel();

    const channel = new Channel(context);
    channel.Send(master);
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

    await pipeline.InitializeDpsPipeline();

    const audioDevice = await pipeline.ResolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const context = audioDevice.GetContext();
    const master = audioDevice.GetMasterChannel();

    const channel = audioDevice.CreateChannel();
    channel.Send(master);
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

    await pipeline.InitializeDpsPipeline();

    const audioDevice = await pipeline.ResolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    const master = audioDevice.GetMasterChannel();

    const channel1 = audioDevice.CreateChannel();
    const channel2 = audioDevice.CreateChannel();
    const channel3 = audioDevice.CreateChannel();

    channel1.Send(channel2);
    channel2.Send(channel3);
    channel3.Send(master);
})()
```