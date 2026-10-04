# Example 06: One-shot sounds

Short sounds that play once: gunshots, impacts, footsteps. There are two patterns:

1. **A persistent source that plays the same clip repeatedly**, such as a gun carried by an enemy.
2. **A temporary source per sound**, such as a bullet impact at a random spot, which is removed when the sound has finished.

## What it shows
- Overlapping playback of one clip with ``overrideMaxAudioBufferNodes`` and ``setMaxAudioBufferSourceNodes``.
- Creating and removing temporary [``SpatialAudioSource``](../classes/SpatialAudioSource.md)s.
- Sounds that belong to the player.

## Setup: allow overlapping playback

By default a clip can only play once at the same time: ``play()`` returns ``null`` while it is still playing. Fast-firing weapons need overlapping playback, which must be enabled on the pipeline:

```ts
const pipeline = new DspPipeline({
    pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
    pathToWorklet: "/bin/fluexgl-dsp-processor.worklet",
    options: {
        overrideMaxAudioBufferNodes: true
    }
});
```

## Pattern 1: a persistent source

```ts
import { AudioClip, AudioSourceData, SpatialAudioRenderer2D, SpatialAudioSource } from "@fluex/fluexgl-dsp";

class Gun {

    public source: SpatialAudioSource;
    private clip: AudioClip;

    constructor(renderer: SpatialAudioRenderer2D, data: AudioSourceData, x: number, y: number) {

        this.source = renderer.createSource({ label: "gun", position: { x, y } });
        this.clip = new AudioClip(data);

        this.source.attachAudioClip(this.clip);

        // Up to 4 shots can ring out at the same time.
        this.clip.setMaxAudioBufferSourceNodes(4);
    }

    public fire() {
        this.clip.play();
    }

    public moveTo(x: number, y: number) {
        this.source.setPosition(x, y);
    }
}
```

## Pattern 2: a temporary source per sound

```ts
function playAt(renderer: SpatialAudioRenderer2D, data: AudioSourceData, x: number, y: number, volume: number = 1) {

    const source = renderer.createSource({ label: "one-shot", position: { x, y }, volume });
    const clip = new AudioClip(data);

    source.attachAudioClip(clip);
    clip.play();

    // Remove the source when the sound has finished. This fades it out of its voice,
    // stops the clip and releases the audio nodes.
    setTimeout(() => renderer.removeSource(source), (clip.duration + 0.25) * 1000);
}

// A burst of impacts far away is clustered automatically into one voice.
for (let i = 0; i < 12; i++)
    playAt(renderer, impactData, 3000 + Math.random() * 200, 400 + Math.random() * 200);
```

A newly created source is rendered immediately, without fading in, so ``play()`` right after ``createSource()`` keeps the attack of the sound. If the source is too far away to be heard, it gets no voice and costs almost nothing.

## Sounds of the player itself

Footsteps, breathing and the player's own weapon are always at the listener's position. Two options:

```ts
// a) Keep them in the spatial renderer, but at the listener's position, so they share the reverb.
const footsteps = renderer.createSource({ label: "footsteps", clusterable: false, airAbsorption: false });
// every frame: footsteps.setPosition(player.x, player.y);

// b) Or play them on a normal channel, outside the spatial renderer.
const playerChannel = audioDevice.createChannel("Player");
playerChannel.send(audioDevice.getMasterChannel());
footstepClip.send(playerChannel);
```

A source exactly at the listener's position is centered (pan ``0``).

## Notes
- Creating an ``AudioClip`` is cheap: clips created from the same ``AudioSourceData`` share the decoded buffer.
- Prefer pattern 1 for sounds that fire often from the same object, and pattern 2 for sounds that happen at random places.
- A sound that should never be merged into a cluster, such as an important voice line, should use ``clusterable: false``.
