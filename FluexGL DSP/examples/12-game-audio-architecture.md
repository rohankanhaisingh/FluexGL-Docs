# Example 12: Game audio architecture

How to organize the audio of a whole game, so it scales to hundreds of objects that make sounds.

A common first approach is to give every object its own [``Channel``](../classes/Channel.md): one for the player, one per NPC, and more for every sound that does not belong to anyone. That quickly adds up to hundreds of channels, which are all processed all the time. This example uses three layers instead:

| Layer | What it is | How many | Cost |
|---|---|---|---|
| **Bus** (``Channel``) | A mixer channel with its own volume and effects | A handful, fixed | A few nodes each |
| **Source** (``SpatialAudioSource``) | A position and a gain, owned by a game object | Hundreds | Two ``GainNode``s |
| **Voice** (internal) | The actual DSP chain: lowpass, panner, reverb send | Limited by ``maxVoices`` | Only for what you can hear |

## What it shows
- Bus channels for effects, entities, UI, ambience and voice chat.
- A spatial renderer that sends to a bus (``output``), and sources on other buses (``bus``).
- Objects with their own source (``source.play()``), fire-and-forget sounds (``renderer.playAt()``) and ambience loops.
- Loop virtualization and ``getStats()``.

## Routing

```
                         +--> Effects bus ----+
SpatialAudioRenderer3D --+--> Entities bus ---+
 (voices per bus,        +--> Ambience bus ---+--> Master [Limiter] --> speakers
  one shared reverb)     +--> Voice chat bus -+
                                              |
sound.play(...) ------------> UI bus ---------+
```

## Code

```ts
import { InputChannel, Limiter, Sound, SpatialAudioRenderer3D } from "@fluex/fluexgl-dsp";

// Assumes `audioDevice` from Example 01.
const master = audioDevice.getMasterChannel();
master.attachEffect(new Limiter({ ceiling: -1 }));

// --- Buses ---
const buses = {
    effects: audioDevice.createChannel("Effects"),
    entities: audioDevice.createChannel("Entities"),
    ui: audioDevice.createChannel("UI"),
    ambience: audioDevice.createChannel("Ambience"),
    voiceChat: audioDevice.createChannel("Voice chat")
};

for (const bus of Object.values(buses))
    bus.send(master);

// --- The world ---
// Everything the renderer plays goes to the Entities bus, unless a source has a bus of its own.
const world = new SpatialAudioRenderer3D(audioDevice, { output: buses.entities });

// --- Sounds ---
const sounds = {
    punch: await Sound.load(audioDevice, "/sfx/punch.wav", { pitchVariation: 1 }),
    gunshot: await Sound.load(audioDevice, "/sfx/gunshot.wav", { maxInstances: 8, pitchVariation: 0.5 }),
    reload: await Sound.load(audioDevice, "/sfx/reload.wav", { maxInstances: 2 }),
    growl: await Sound.load(audioDevice, "/sfx/growl.wav", { maxInstances: 6, steal: "quietest" }),
    explosion: await Sound.load(audioDevice, "/sfx/explosion.wav", { maxInstances: 4, steal: "quietest" }),
    impact: await Sound.load(audioDevice, "/sfx/impact.wav", { minInterval: 0.03, volumeVariation: 0.3 }),
    torch: await Sound.load(audioDevice, "/sfx/torch-loop.wav", { loop: true, maxInstances: Infinity }),
    click: await Sound.load(audioDevice, "/sfx/click.wav")
};
```

### The player

The player is the listener, so its own sounds are not positioned. They are played directly on the Effects bus.

```ts
function onPlayerShoot() {
    sounds.gunshot.play(buses.effects);
}

function onPlayerReload() {
    sounds.reload.play(buses.effects);
}
```

### NPCs and other entities

Every entity gets one source, not a channel. All of its sounds play through that source, so they move with it.

```ts
class Npc {

    public x = 0;
    public y = 0;
    public z = 0;

    private source = world.createSource({ label: "npc", bus: buses.entities });

    public update() {
        this.source.setPosition(this.x, this.y, this.z);
    }

    public attack() {
        this.source.play(sounds.punch);
    }

    public growl() {
        this.source.play(sounds.growl);
    }

    public destroy() {
        world.removeSource(this.source); // fades out and stops its sounds
    }
}
```

### Sounds that belong to nobody

Explosions far away, objects that collide, bullet impacts: play them where they happen and forget about them.

```ts
function onExplosion(position) {
    world.playAt(sounds.explosion, position, { bus: buses.effects, refDistance: 10, maxDistance: 400 });
}

function onCollision(a, b, contactPoint, strength) {
    world.playAt(sounds.impact, contactPoint, { bus: buses.effects, volume: Math.min(1, strength) });
}
```

Collisions far away are culled (not played at all), and ``minInterval`` keeps a pile of colliding objects from playing the same sound 40 times in one frame.

### Ambience of objects

A torch, a waterfall or a generator is a loop at a fixed position.

```ts
for (const torch of level.torches)
    world.playAt(sounds.torch, torch.position, { bus: buses.ambience, loop: true, volume: 0.5 });
```

A level can have hundreds of them. Loops that nobody can hear (too far away, or out of the voice budget) are suspended after half a second: their audio nodes are released, and they resume in time when you come closer. See [Loop virtualization](../classes/SpatialAudioRenderer.md#loop-virtualization).

Keep the returned [``SoundInstance``](../classes/SoundInstance.md) if the loop must stop later, for example when the torch is put out:

```ts
torch.sound = world.playAt(sounds.torch, torch.position, { bus: buses.ambience, loop: true });
// later:
torch.sound?.stop(0.5);
```

### Voice chat

Every remote player gets an [``InputChannel``](../classes/InputChannel.md) with their stream, positioned by a source on the Voice chat bus. See [Example 11](./11-proximity-voice-chat.md) for the full setup.

```ts
function onRemotePlayerJoined(player, stream: MediaStream) {
    const voice = new InputChannel(audioDevice.getContext(), player.name);
    voice.setMediaStream(stream);

    player.voiceSource = world.createSource({ bus: buses.voiceChat, clusterable: false });
    player.voiceSource.attachChannel(voice);
}
```

### UI

```ts
button.addEventListener("click", () => sounds.click.play(buses.ui));
```

### The game loop and the settings menu

```ts
function frame() {
    // See Example 05 for following a three.js camera, including its orientation.
    world.listener.setPosition(camera.position.x, camera.position.y, camera.position.z);

    for (const npc of npcs) npc.update();

    world.update();
    requestAnimationFrame(frame);
}

// Volume sliders: one per bus.
settings.onChange("effects", (v) => buses.effects.volume(v));
settings.onChange("voiceChat", (v) => buses.voiceChat.volume(v));
```

## Debugging

```ts
const stats = world.getStats();
debugText.textContent = `sources ${stats.sources}, voices ${stats.voices}/${world.options.maxVoices}, ` +
    `virtual ${stats.virtual}, suspended loops ${stats.suspendedLoops}`;
```

## Notes
- Sources only share a voice (cluster) with sources on the same bus, so a bus volume never affects another bus. The reverb is shared, and returns into the renderer's ``output`` (here the Entities bus).
- With an ``output``, the renderer does not add a limiter of its own. Put one on the master channel, as above, or pass ``limiter: true``.
- ``world.master`` is ``null`` when the output is a ``Channel``. Use the buses for volumes instead.
- Effects that should apply to a whole category, such as a low-pass filter on everything except the UI while the game is paused, go on the buses.
