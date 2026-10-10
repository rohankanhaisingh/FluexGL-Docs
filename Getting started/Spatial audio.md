# Spatial audio

FluexGL DSP can place sounds in a 2D or 3D world. Sounds get quieter, duller and more reverberant the further they are from the listener, and are panned depending on their direction. Far away sounds in the same direction are clustered, so a battle with fifty guns does not need fifty filters and panners.

This page explains the concepts. For complete code, see the [examples](../FluexGL%20DSP/examples/README.md).

---

## The three building blocks

| Class | What it is |
|---|---|
| [``SpatialAudioRenderer2D``](../FluexGL%20DSP/classes/SpatialAudioRenderer2D.md) / [``SpatialAudioRenderer3D``](../FluexGL%20DSP/classes/SpatialAudioRenderer3D.md) | Owns everything: its own master channel (or a bus you pass as ``output``), a reverb bus, a limiter, one listener and all sources. Call ``update()`` every frame. |
| [``SpatialAudioListener``](../FluexGL%20DSP/classes/SpatialAudioListener.md) / [``SpatialAudioListener3D``](../FluexGL%20DSP/classes/SpatialAudioListener3D.md) | The "ears" of the scene, usually the player or the camera. Every renderer has exactly one. |
| [``SpatialAudioSource``](../FluexGL%20DSP/classes/SpatialAudioSource.md) | A sound emitter in the world, owned by a game object. Play [``Sound``](../FluexGL%20DSP/classes/Sound.md)s on it with ``source.play()``, attach ``AudioClip``s or ``Channel``s (such as a voice on an [``InputChannel``](../FluexGL%20DSP/classes/InputChannel.md)), and move it with your game object. For voice chat, see [Example 11](../FluexGL%20DSP/examples/11-proximity-voice-chat.md). |

```ts
const renderer = new SpatialAudioRenderer2D(audioDevice);
const source = renderer.createSource({ position: { x: 300, y: 200 } });

source.attachAudioClip(clip);
clip.play();

function frame() {
    renderer.listener.setPosition(player.x, player.y);
    source.setPosition(enemy.x, enemy.y);
    renderer.update();
    requestAnimationFrame(frame);
}
```

---

## What distance does

For every source, on every ``update()``:

| Effect | Controlled by | Behavior |
|---|---|---|
| Volume | ``distanceModel``, ``refDistance``, ``maxDistance``, ``rolloffFactor`` | Full volume within ``refDistance``, silent at ``maxDistance``. Same formulas as the Web Audio ``PannerNode``. |
| Lowpass (air absorption) | ``lowpassMaxFrequency``, ``lowpassMinFrequency`` | Cutoff moves logarithmically from 20 kHz to 600 Hz between ``refDistance`` and ``maxDistance``. |
| Rear lowpass | ``rearLowpassFactor`` | Sources behind the listener sound slightly duller (cutoff x 0.6). |
| Reverb | ``reverbMinSend``, ``reverbMaxSend`` | The reverb send grows from 0.05 to 0.6 with distance. |
| Panning | listener orientation | Left/right in 2D. Left/right, front/back and up/down in 3D with HRTF. |

Sources that are too quiet to hear (below ``silenceThreshold``) get no voice at all, so they cost almost nothing.

The renderer settings are the defaults for every source. A source can override the attenuation settings:

```ts
renderer.createSource({ position, refDistance: 200, maxDistance: 4000 }); // a loud explosion
```

---

## Voices and clustering

A **voice** is the expensive part of the chain: a lowpass filter, a panner and a reverb send. Every source always keeps its own gain, so its loudness is exact, but a voice can be shared.

- **Close sources** (closer than ``clustering.splitDistance``) always have their own voice.
- **Far sources** (further than ``clustering.mergeDistance``) in roughly the same direction (within ``maxAngle``, default 15 degrees) and at a similar distance (within ``maxDistanceRatio``, default 35%) share one voice: a **cluster**.
- When the listener walks towards a cluster, it splits up again.

The gap between ``splitDistance`` and ``mergeDistance`` (hysteresis) prevents sources from switching back and forth. Moving a source between voices always uses a short crossfade, so there are no clicks.

```ts
const renderer = new SpatialAudioRenderer2D(audioDevice, {
    clustering: { splitDistance: 250, mergeDistance: 340, maxAngle: Math.PI / 10 }
});

renderer.clustering.enabled = false; // can be changed at any time
```

Set ``clusterable: false`` on a source that should always keep its own voice, such as dialogue.

---

## Sounds in a game world

There are three ways to play a [``Sound``](../FluexGL%20DSP/classes/Sound.md), depending on who it belongs to:

```ts
npcSource.play(sounds.growl);                       // an object with its own source: moves with it
renderer.playAt(sounds.explosion, { x: 900, y: 120 }); // nobody: fire-and-forget at a position
sounds.click.play(uiBus);                           // not positioned: UI, the player itself
```

``playAt()`` borrows a source from a pool and returns it when the sound has ended. One-shots that are too far away to be heard are not played at all.

Looping sounds (a torch, a waterfall) whose source has no voice for half a second are suspended: their audio nodes are released, and they continue in time once you come closer. See [Loop virtualization](../FluexGL%20DSP/classes/SpatialAudioRenderer.md#loop-virtualization).

## Buses

A renderer sends everything to its own master channel, unless you give it an ``output``. Sources can be routed to another bus with ``bus``:

```ts
const world = new SpatialAudioRenderer3D(audioDevice, { output: entitiesBus });

world.createSource({ position, bus: ambienceBus });
world.playAt(sounds.explosion, position, { bus: effectsBus });
```

Sources only share a voice with sources on the same bus. See [Example 12: Game audio architecture](../FluexGL%20DSP/examples/12-game-audio-architecture.md).

---

## 2D or 3D?

| | 2D | 3D |
|---|---|---|
| Class | ``SpatialAudioRenderer2D`` | ``SpatialAudioRenderer3D`` |
| Panner | ``StereoPannerNode`` (left/right) | ``PannerNode`` with ``"HRTF"`` (default), ``"equalpower"`` or ``"stereo"`` |
| Coordinates | Screen space, y down by default (``yAxis: "up"`` for math coordinates) | Right-handed, y up, -z forward (Web Audio / three.js) |
| Default units | Pixels: ``refDistance: 50``, ``maxDistance: 1500`` | Meters: ``refDistance: 1``, ``maxDistance: 100`` |
| Listener orientation | Optional ``rotation`` (radians, clockwise, 0 = facing up on screen) | ``setOrientation(forward, up)``, ``setYawPitch()`` or ``lookAt()`` |

### Rotation in 2D is optional
With the default rotation of ``0``, left and right on the screen are left and right in your ears. That is what most 2D games want. Only set a rotation when the player actually turns (for example in a top-down shooter):

```ts
renderer.listener.lookAt(mouse.x, mouse.y);
```

In a side-scroller, sources below the player are not "behind" the player. Disable the rear lowpass with ``rearLowpassFactor: 1``.

### HRTF in 3D
HRTF gives the brain front/back and elevation cues. It works best on headphones and is more expensive than stereo panning, which is where clustering helps: far away sources share one HRTF panner.

---

## Loudness: the limiter

Every spatial renderer has a [``Limiter``](../FluexGL%20DSP/effects/Limiter.md) on its master channel (ceiling -1 dB), so many sources at once do not clip. Configure or disable it with the ``limiter`` option. A renderer with an ``output`` has no limiter unless you pass ``limiter: true`` (or options); put one on your master channel instead.

```ts
new SpatialAudioRenderer3D(audioDevice, { limiter: { ceiling: -3 } });
new SpatialAudioRenderer3D(audioDevice, { limiter: false });
```

---

## Reverb

Every renderer has a reverb bus (``renderer.reverbChannel``). The default [``Reverb``](../FluexGL%20DSP/effects/Reverb.md) runs on an AudioWorklet, so it needs WebAssembly. It is attached automatically on the first ``update()`` after the DSP pipeline is initialized. Before that, the bus is muted.

Use your own reverb, or none:

```ts
renderer.setReverbEffect(new Reverb({ mix: 1, roomSize: 0.9, damping: 0.2 }));
renderer.setReverbEffect(null);
```

---

## Debugging

```ts
// Where is a source, as heard by the listener?
console.log(source.state); // { distance, azimuth, elevation, attenuation, ... }

// Which sources share a voice?
console.log(renderer.getClusters());

// What is a voice doing right now?
console.log(source.voice?.filter.frequency.value, source.voice?.isCluster);

// Voices, virtual sources and suspended loops at a glance.
console.log(renderer.getStats());
```

See [Debugging and visualizing clusters](../FluexGL%20DSP/examples/07-debugging-clusters.md).
