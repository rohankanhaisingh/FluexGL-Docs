# Example 06: One-shot sounds

Short sounds that play once: gunshots, impacts, footsteps, explosions. In a game these are played often, from many places at once, so they should be cheap. Use [``Sound``](../classes/Sound.md) for them: one decoded buffer, played as instances that are released as soon as they have finished.

There are three patterns:

1. **Sounds of an object that has its own source**, such as a gun carried by an enemy: ``source.play(sound)``.
2. **Sounds at a position that do not belong to an object**, such as a bullet impact or an explosion: ``renderer.playAt(sound, position)``.
3. **Sounds of the player itself and UI sounds**, which are not positioned: ``sound.play(bus)``.

## What it shows
- Loading sounds with ``Sound.load()``.
- Limiting overlapping instances (``maxInstances``, ``steal``) and skipping duplicate starts (``minInterval``).
- Random pitch and volume variation.
- Fire-and-forget sounds with ``renderer.playAt()``.

## Setup: loading the sounds

```ts
import { Sound, SpatialAudioRenderer2D } from "@fluex/fluexgl-dsp";

// Assumes `audioDevice` from Example 01.
const sounds = {
    // Up to 6 shots ring out at the same time; the 7th stops the oldest one.
    gunshot: await Sound.load(audioDevice, "/sfx/gunshot.wav", { maxInstances: 6, pitchVariation: 0.5 }),

    // 40 collisions in the same frame should not play 40 times.
    impact: await Sound.load(audioDevice, "/sfx/impact.wav", { minInterval: 0.03, volumeVariation: 0.3 }),

    // Far away explosions make room for close ones.
    explosion: await Sound.load(audioDevice, "/sfx/explosion.wav", { maxInstances: 4, steal: "quietest" }),

    footstep: await Sound.load(audioDevice, "/sfx/step.wav", { volume: 0.6, pitchVariation: 1, volumeVariation: 0.25 })
};

const renderer = new SpatialAudioRenderer2D(audioDevice);
renderer.start();
```

``Sound.load()`` decodes on the device's ``AudioContext`` and caches the result per url, so loading the same file twice does not decode it twice.

## Pattern 1: an object with its own source

```ts
import { SpatialAudioSource } from "@fluex/fluexgl-dsp";

class Gun {

    public source: SpatialAudioSource;

    constructor(renderer: SpatialAudioRenderer2D, x: number, y: number) {
        this.source = renderer.createSource({ label: "gun", position: { x, y } });
    }

    public fire() {
        this.source.play(sounds.gunshot);
    }

    public moveTo(x: number, y: number) {
        this.source.setPosition(x, y);
    }
}
```

Every ``fire()`` starts a new instance on the same source. The number of gunshots playing at the same time is limited by ``maxInstances`` of the sound, across all guns.

## Pattern 2: fire-and-forget at a position

```ts
// A bullet hits a wall.
renderer.playAt(sounds.impact, { x: hit.x, y: hit.y });

// A burst of impacts far away is clustered automatically into one voice.
for (let i = 0; i < 12; i++)
    renderer.playAt(sounds.impact, { x: 3000 + Math.random() * 200, y: 400 + Math.random() * 200 }, { cull: false });

// An explosion that can be heard from further away than the default.
renderer.playAt(sounds.explosion, { x: 900, y: 120 }, { refDistance: 150, maxDistance: 4000 });
```

``playAt()`` borrows a source from a pool and returns it when the sound has ended, so you do not have to create or remove anything. A newly started sound is rendered immediately, without fading in, so its attack stays intact. A one-shot that is too far away to be heard when it starts is not played at all and ``playAt()`` returns ``null``, unless ``cull: false`` is passed.

## Pattern 3: the player and the UI

Footsteps, breathing and the player's own weapon are always at the listener's position. Two options:

```ts
// a) Not positioned, on a channel of its own (outside the spatial renderer).
const playerChannel = audioDevice.createChannel("Player");
playerChannel.send(audioDevice.getMasterChannel());

sounds.footstep.play(playerChannel);

// b) Or in the spatial renderer, at the listener's position, so they share the reverb.
const playerSource = renderer.createSource({ label: "player", clusterable: false, airAbsorption: false });
// every frame: playerSource.setPosition(player.x, player.y);

playerSource.play(sounds.footstep);
```

A source exactly at the listener's position is centered (pan ``0``).

## Controlling an instance

```ts
const instance = renderer.playAt(sounds.explosion, { x: 900, y: 120 });

instance?.onEnded(() => console.log("done"));
instance?.setVolume(0.5);
instance?.stop(0.1); // fade out in 100 ms
```

## Notes
- ``sound.play()``, ``source.play()`` and ``playAt()`` return ``null`` when a start was skipped (``minInterval``, ``maxInstances`` or culled). Use optional chaining on the result.
- Prefer pattern 1 for sounds that come from an object, and pattern 2 for sounds that happen at random places.
- A sound that should never be merged into a cluster, such as an important voice line, should use ``clusterable: false`` (on the source, or in the ``playAt()`` options).
- [``AudioClip``](../classes/AudioClip.md) is still the right choice for music and for sounds you want to seek or follow the progress of. For overlapping clip playback see [Multiple audio buffer sources](../../Tips/Multiple%20audio%20buffer%20sources.md).
- For looping sounds in the world and for organizing a whole game into buses, see [Example 12](./12-game-audio-architecture.md).
