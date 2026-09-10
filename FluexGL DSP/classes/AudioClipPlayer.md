# Class ``AudioClipPlayer``

Plays and manages [``AudioClip``](./AudioClip.md) instances inside a [``Channel``](./Channel.md).

## Example

```ts
import { AudioClipPlayer, AudioClip, Channel } from "@fluex/fluexgl-dsp";

...

const context = audioDevice.getContext();
const channel = audioDevice.createChannel();

const player = new AudioClipPlayer(context);
const clip = new AudioClip(audioSourceData);

player.attachAudioClip(clip);
player.send(channel);
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
Internal output gain node. This is where volume is applied and what gets connected to a target via ``send()``.

### ``context: AudioContext | null``
The ``AudioContext`` used to create this player’s internal nodes. Set during construction.

### ``channel: Channel | Master | null``
The current send target ([``Channel``](./Channel.md) or [``Master``](./Master.md)) this player is connected to. ``null`` when not routed.

- - -

## Methods

### ``attachAudioClip(audioClip: AudioClip): void``
Attaches an [``AudioClip``](./AudioClip.md) to this player. Initializes the clip with this player and stores it in ``audioClips``. Logs an error if the clip is already attached.

#### Arguments
- ``audioClip``: [``AudioClip``](./AudioClip.md) - The clip to attach.

#### Returns
- ``void``

### ``detachAudioClip(clip: AudioClip): void``
Detaches a previously attached [``AudioClip``](./AudioClip.md) from this player. Logs an error if the clip is not attached.

#### Arguments
- ``clip``: [``AudioClip``](./AudioClip.md) - The clip to detach.

#### Returns
- ``void``

### ``send(channel: Channel | Master): void``
Routes this player’s ``outputGainNode`` to a [``Channel``](./Channel.md) or the [``Master``](./Master.md) by connecting to the target’s ``input`` node.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The target to receive this player’s signal.

#### Returns
- ``void``

### ``unsend(): void``
Disconnects this player’s ``outputGainNode`` from its current target (if any) and clears ``channel``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``setVolume(value: number): void``
Sets the output volume by writing to ``outputGainNode.gain.value``. The value is clamped between ``0`` and ``1``.

#### Arguments
- ``value``: ``number`` - Volume from 0.0 (silent) to 1.0 (full scale). Values outside the range are clamped.

#### Returns
- ``void``

### ``stopAll(): void``
Stops playback for all attached clips by calling ``stop()`` on each clip in ``audioClips``.

#### Arguments
No arguments

#### Returns
- ``void``

### ``dispose(): void``
Stops all clips, disconnects routing, disconnects the output node, and clears internal references (clips, nodes, context, channel). Use when you no longer need this player.

#### Arguments
No arguments

#### Returns
- ``void``

### ``setLabel(label: string): void``
Updates the ``label`` of this player.

#### Arguments
- ``label``: ``string`` - New label.

#### Returns
- ``void``

## Events

This class does not emit custom events.

- - -

## Getters and setters

### ``get length(): number``
Returns the number of [``AudioClip``](./AudioClip.md) instances currently attached to this player (``audioClips.length``).

### ``get volume(): number``
Returns the current output volume, read from ``outputGainNode.gain.value``. Returns ``0`` when no output gain node is available.

- - -

## Examples

### Example: attaching, routing, and stopping clips
```ts
import { AudioClip, Channel } from "@fluex/fluexgl-dsp";

const context = new AudioContext();
const channel = new Channel(context);

// Use the Channel-owned player in real usage
const clip = new AudioClip(audioSourceData);
channel.attachAudioClip(clip);

// Stop everything routed through the channel’s player
channel.audioClipPlayer.stopAll();
```
