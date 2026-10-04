# Example 08: Mixing and loudness

A typical game mix: music, the spatial world and UI sounds, each with their own volume, and nothing that clips.

## What it shows
- Combining a [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) or [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md) with normal [``Channel``](../classes/Channel.md)s.
- The renderer's built-in [``Limiter``](../effects/Limiter.md), and a limiter on the device's master channel.
- A [``Compressor``](../effects/Compressor.md) on the music.
- Ducking the world volume.

## Routing

```
Music clip -> Channel "Music" [Compressor] --+
UI clips   -> Channel "UI" ------------------+--> device master [Limiter] --> speakers
                                                                               ^
World sources -> SpatialAudioRenderer -> renderer.master [Limiter] ------------+
```

Every spatial renderer has its own master channel, created with ``audioDevice.createMasterChannel()``. It connects to the speakers next to the device's default master channel.

## Code

```ts
import { AudioClip, Compressor, Limiter, SpatialAudioRenderer2D } from "@fluex/fluexgl-dsp";

// Assumes `audioDevice` from Example 01, and loaded `musicData` and `clickData`.
const deviceMaster = audioDevice.getMasterChannel();

// --- Music: compressed, not spatial ---
const music = audioDevice.createChannel("Music");
music.send(deviceMaster);
music.addEffect(new Compressor({ threshold: -18, ratio: 3, makeupGain: 2 }));
music.volume(0.5);

const musicClip = new AudioClip(musicData);
musicClip.send(music);
musicClip.setLoop(true);
musicClip.play();

// --- UI: not spatial ---
const ui = audioDevice.createChannel("UI");
ui.send(deviceMaster);

const clickClip = new AudioClip(clickData);
clickClip.send(ui);

// --- Safety limiter on the device master (music + UI) ---
deviceMaster.attachEffect(new Limiter({ ceiling: -1 }));

// --- World: spatial, with its own master and limiter ---
const world = new SpatialAudioRenderer2D(audioDevice, {
    limiter: { ceiling: -1, release: 0.15 }
});

// World volume, for example from a settings menu (0..1).
function setWorldVolume(volume: number) {
    world.master.gainNode?.gain.setTargetAtTime(volume, audioDevice.context.currentTime, 0.05);
}

// Duck the world while a dialog is open.
function setDialogOpen(open: boolean) {
    setWorldVolume(open ? 0.3 : 1);
}
```

## Metering

```ts
setInterval(() => {
    console.log("world limiter reduction:", world.limiter?.reduction.toFixed(1), "dB");
}, 250);
```

If the limiter is reducing by more than a few dB most of the time, the mix is too loud: lower source volumes or ``world.master`` instead of relying on the limiter.

## Notes
- A renderer's ``master`` is a normal [``Master``](../classes/Master.md). Its ``gainNode`` sits after the limiter, so lowering it never causes clipping.
- Effects attached to ``world.master`` after construction come after the built-in limiter. If you need effects before the limiter, create the renderer with ``limiter: false``, attach your effects, then attach your own ``Limiter`` last.
- To mute a channel, use ``channel.volume(0)``.
- Two renderers can run side by side, for example one for the world and one for a separate minigame. They can share one ``AudioDevice``.
