# Class ``AudioClipPlayer``

Plays and manages [``AudioClip``](./AudioClip.md) instances inside a [``Channel``](./Channel.md).

## Example

```ts
import { AudioClipPlayer, AudioClip, Channel } from "@fluex/fluexgl-dsp";

...

const context = audioDevice.GetContext();
const channel = audioDevice.CreateChannel();

const player = new AudioClipPlayer(context);
const clip = new AudioClip(audioSourceData);

player.AttachAudioClip(clip);
player.Send(channel);
```

- - -

## Constructor
Constructs a new AudioClipPlayer and wires it into the provided Channel.

```ts
new AudioClipPlayer(context: AudioContext): AudioClipPlayer;
```

### Arguments
- ``context``: ``AudioContext`` - The AudioContext used to create this player’s internal audio nodes.

- - -

## Properties

### ``label: string``
A human-readable label for debugging / UI purposes. Defaults to ``"AudioClipPlayer"``.

### ``id: string``
A unique id, automatically generated on construction. Should NOT be changed.

### ``audioClips: AudioClip[]``
All [``AudioClip``](./AudioClip.md) instances currently attached to this player.

### ``outputGainNode: GainNode | null``
Internal output gain node. This is where volume is applied and what gets connected to a target via ``Send()``.

### ``context: AudioContext | null``
The ``AudioContext`` used to create this player’s internal nodes. Set during construction.

### ``channel: Channel | Master | null``
The current send target ([``Channel``](./Channel.md) or [``Master``](./Master.md)) this player is connected to. ``null`` when not routed.

- - -

## Methods

### ``AttachAudioClip(audioClip: AudioClip): void``
Attaches an [``AudioClip``](./AudioClip.md) to this player. Initializes the clip with this player and stores it in ``audioClips``. Throws if the clip is already attached.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md) - The clip to attach.

#### Returns
- ``void``

### ``DetachAudioClip(clip: AudioClip): void``
Detaches a previously attached [``AudioClip``](./AudioClip.md) from this player. Throws if the clip is not attached.

#### Arguments
- ``clip``: [``AudioClip``](./AudioClip.md) - The clip to detach.

#### Returns
- ``void``

### ``Send(channel: Channel | Master): void``
Routes this player’s ``outputGainNode`` to a [``Channel``](./Channel.md) or the [``Master``](./Master.md) by connecting to the target’s ``input`` node.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target to receive this player’s signal.

#### Returns
- ``void``

### ``Unsend(): void``
Disconnects this player’s ``outputGainNode`` from its current target (if any) and clears ``channel``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``SetVolume(value: number): void``
Sets the output volume by writing to ``outputGainNode.gain.value``. The value is clamped between ``0`` and ``1``.

#### Arguments
- ``value``: ``number`` - Volume from 0.0 (silent) to 1.0 (full scale). Values outside the range are clamped.

#### Returns
- ``void``

### ``StopAll(): void``
Stops playback for all attached clips by calling ``Stop()`` on each clip in ``audioClips``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``Dispose(): void``
Stops all clips, disconnects routing, disconnects the output node, and clears internal references (clips, nodes, context, channel). Use when you no longer need this player.

#### Arguments
No arguments

#### Returns
- ``void``

### ``SetLabel(label: string): void``
Updates the ``label`` of this player.

#### Arguments
- ``label``: ``string`` - New label.

#### Returns
- ``void``

## Events

This class does not emit custom events.

- - -

## Getters and setters

This class does not define public getters or setters.

- - -

## Examples

### Example: attaching, routing, and stopping clips
```ts
import { AudioClip, Channel } from "@fluex/fluexgl-dsp";

const context = new AudioContext();
const channel = new Channel(context);

// Use the Channel-owned player in real usage
const clip = new AudioClip(audioSourceData);
channel.AttachAudioClip(clip);

// Stop everything routed through the channel’s player
channel.audioClipPlayer.StopAll();
```
