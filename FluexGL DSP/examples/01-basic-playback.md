# Example 01: Basic playback

Initializes the DSP pipeline, routes a clip through a channel into the master channel, and plays it.

## What it shows
- Initializing [``DspPipeline``](../classes/DspPipeline.md) and resolving an [``AudioDevice``](../classes/AudioDevice.md).
- Creating a [``Channel``](../classes/Channel.md) and sending it to the [``Master``](../classes/Master.md).
- Loading audio with [``loadAudioSource``](../helpers/loadAudioSource.md) and playing an [``AudioClip``](../classes/AudioClip.md).

## Routing

```
AudioClip -> Channel "Music" -> Master -> speakers
```

## Code

```ts
import { DspPipeline, AudioClip, loadAudioSource } from "@fluex/fluexgl-dsp";

async function start() {

    // 1. Initialize the pipeline. This compiles the WebAssembly module and prepares the worklet.
    //    Note: it asks for microphone permission (getUserMedia) to be able to list audio devices.
    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    if (!await pipeline.initializeDpsPipeline()) return;

    // 2. Resolve the default output device. This also loads the worklet on its AudioContext.
    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    // Browsers only start audio after a user gesture.
    await audioDevice.context.resume();

    // 3. Routing: channel -> master.
    const master = audioDevice.getMasterChannel();
    const music = audioDevice.createChannel("Music");

    music.send(master);
    music.volume(0.8);

    // 4. Load and play a clip.
    const data = await loadAudioSource("/audio/song.ogg");

    if (!data) return;

    const clip = new AudioClip(data);

    clip.send(music);
    clip.setLoop(true);
    clip.play();

    // Optional: follow the playback position.
    clip.addEventListener("progress", (event) => {
        console.log(`${event.formatted} / ${clip.formattedDuration}`);
    });
}

// Start on the first click, because of the browser's autoplay policy.
document.addEventListener("click", start, { once: true });
```

## Controlling the clip

```ts
clip.setVolume(0.5);   // gain of this clip
clip.setPanLevel(-0.3); // -1 (left) to 1 (right)
clip.setPitch(2);      // semitones, -24 to 24
clip.seek(30);         // seconds
clip.stop();
```

## Notes
- ``initializeDpsPipeline`` is spelled with ``Dps``. ``init()`` is a shorthand.
- One ``AudioClip`` plays into every channel it was sent to. To play the same sound in two places independently, create two clips from the same ``AudioSourceData``; they share the decoded buffer.
- By default, a clip can only play once at the same time: ``play()`` returns ``null`` while it is still playing. See [Example 06](./06-one-shot-sounds.md) for overlapping sounds.
