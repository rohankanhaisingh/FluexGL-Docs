# Class ``Master``

The master bus of an [``AudioDevice``](./AudioDevice.md): the final routing point that every [``Channel``](./Channel.md) eventually connects into, and which connects to ``AudioContext.destination``. Can also host its own effect chain and directly attached [``AudioClip``](./AudioClip.md) instances via an internal [``AudioClipPlayer``](./AudioClipPlayer.md).

## Example

```ts
import { Master, Channel } from "@fluex/fluexgl-dsp";

const master = new Master(context);
const channel = new Channel(context);

channel.send(master);
```

- - -

## Constructor
Constructs a new Master channel, wiring ``input`` → effects → ``gainNode`` → ``analyserNode`` → ``context.destination``, and creates its internal [``AudioClipPlayer``](./AudioClipPlayer.md).

```ts
new Master(context: AudioContext): Master;
```

### Arguments
- ``context``: ``AudioContext`` - The AudioContext used to create this master channel's internal audio nodes.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing a new master channel. Should NOT be changed.

### ``channels: Channel[]``
All [``Channel``](./Channel.md) instances currently attached to this master via ``attachChannel()``.

### ``effects: Effector[]``
List of attached [``Effector``](./Effector.md) instances, wired between ``input`` and ``gainNode``.

### ``input: GainNode | null``
Input node of this master channel. Channels are connected here (directly, or through the effect chain) when attached.

### ``gainNode: GainNode | null``
Gain node used to control the overall output volume of this master channel.

### ``analyserNode: AnalyserNode | null``
Analyser node used for visualization / analysis of the master signal, placed right before ``context.destination``.

### ``context: AudioContext | null``
The AudioContext this master channel was constructed with.

### ``audioClipPlayer: AudioClipPlayer | null``
[``AudioClipPlayer``](./AudioClipPlayer.md) owned by this master channel. Used to attach and play [``AudioClip``](./AudioClip.md) instances directly into the master bus.

- - -

## Methods

### ``attachEffect(effect: Effector): void``
Adds an [``Effector``](./Effector.md) to this master channel, initializes it using this channel's ``AudioContext``, and rebuilds the internal effect chain routing. Logs an error if the effect is already attached.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to attach.

#### Returns
- ``void``

### ``detachEffect(effect: Effector): void``
Removes an attached [``Effector``](./Effector.md) from this master channel and rebuilds the internal effect chain routing. Logs an error if the effect is not part of this master channel.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) - The effect instance to detach.

#### Returns
- ``void``

### ``attachChannel(channel: Channel): void``
Registers a [``Channel``](./Channel.md) as connected into this master channel's ``input``. Logs an error if the channel is already attached. This is normally called for you via ``channel.send(master)``.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) - The channel to attach.

#### Returns
- ``void``

### ``detachChannel(channel: Channel): void``
Disconnects a previously attached [``Channel``](./Channel.md) from this master channel's ``input``. Logs an error if the channel is not attached. This is normally called for you via ``channel.unsend(master)``.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) - The channel to detach.

#### Returns
- ``void``

### ``hasAudioClipPlayer(): boolean``
Returns whether this master channel has a constructed [``AudioClipPlayer``](./AudioClipPlayer.md).

#### Arguments
No arguments

#### Returns
- ``boolean`` - ``true`` if ``audioClipPlayer`` is defined, otherwise ``false``.

### ``attachAudioClip(audioClip: AudioClip): void``
Attaches an [``AudioClip``](./AudioClip.md) directly to this master channel via its internal [``AudioClipPlayer``](./AudioClipPlayer.md). Logs an error if this master channel has no ``audioClipPlayer``.

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

### Example 1: getting the default master channel from a device
```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

(async function () {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet"
    });

    await pipeline.initializeDpsPipeline();

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();
    if (!audioDevice) return;

    const master = audioDevice.getMasterChannel();
    const channel = audioDevice.createChannel();

    channel.send(master);
})();
```

### Example 2: playing an audio clip directly through the master
```ts
import { AudioClip } from "@fluex/fluexgl-dsp";

const master = audioDevice.getMasterChannel();
const clip = new AudioClip(audioSourceData);

clip.send(master);
clip.play();
```
