# Example 04: Spatial 2D, side-scroller

A side-scroller (platformer) where the camera follows the player. The listener never rotates: left/right on screen is left/right in the speakers.

## What it shows
- A [``SpatialAudioRenderer2D``](../classes/SpatialAudioRenderer2D.md) without listener rotation.
- ``rearLowpassFactor: 1``: in a side-scroller, sounds below the player are not "behind" the player.
- ``yAxis`` for engines that use y-up coordinates.
- Per-source attenuation: a waterfall that can be heard from far away.

## Code

```ts
import { AudioClip, SpatialAudioRenderer2D } from "@fluex/fluexgl-dsp";

// Assumes `audioDevice` from Example 01, and loaded `waterfallData` and `birdData`.
const renderer = new SpatialAudioRenderer2D(audioDevice, {
    // Most 2D engines and the canvas use y-down. Use "up" if your world uses y-up.
    yAxis: "down",

    // No "behind" in a side-scroller.
    rearLowpassFactor: 1,

    // The screen is about 1920 px wide; sounds off-screen fade out over the next screen.
    refDistance: 100,
    maxDistance: 2400
});

// The waterfall is loud: it overrides the attenuation of the renderer for this source only.
const waterfall = renderer.createSource({
    label: "waterfall",
    position: { x: 5200, y: 300 },
    refDistance: 400,
    maxDistance: 6000
});

const waterfallClip = new AudioClip(waterfallData);
waterfall.attachAudioClip(waterfallClip);
waterfallClip.setLoop(true);
waterfallClip.play();

// Birds in a tree: no reverb, they are outside.
const birds = renderer.createSource({
    label: "birds",
    position: { x: 2600, y: -200 },
    reverbSendFactor: 0
});

const birdClip = new AudioClip(birdData);
birds.attachAudioClip(birdClip);
birdClip.setLoop(true);
birdClip.play();

function loop() {

    // ... update the player and camera ...

    // The listener follows the player. No rotation needed.
    renderer.listener.setPosition(player.x, player.y);
    renderer.update();

    requestAnimationFrame(loop);
}

requestAnimationFrame(loop);
```

## Notes
- Should the listener be the player or the camera? Usually the player: sounds then stay balanced around the character. Use the camera centre when the camera leads far ahead of the player.
- Going inside a cave? Change the reverb at runtime: ``renderer.setReverbEffect(new Reverb({ mix: 1, roomSize: 0.95, damping: 0.2 }))``.
- Sounds that are not part of the world (music, UI) do not belong in the renderer. See [Example 08](./08-mixing-and-loudness.md).
