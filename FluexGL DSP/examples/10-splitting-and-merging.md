# Example 10: Splitting and merging (pseudo surround)

Splits a stereo signal into its mid and side parts, processes the side into a diffuse "rear", and merges both parts again. The result sounds wider and more surrounding, especially on headphones.

## What it shows
- Splitting a channel with [``StereoMono.split()``](../effects/StereoMono.md).
- Processing one branch on its own: delay, polarity, filter and volume.
- Merging by sending both branches to the same channel.
- Bypassing the split at runtime with ``isSentTo()``, ``send()`` and ``unsend()``.

## Routing

```
                                  +-> "Music Mid"  ----------------------------------+
AudioClip -> Channel "Music" -----+                                                  +-> Channel "Surround" -> Master
                                  +-> "Music Side" -> delay -> LowPassFilter -> x1.4 +
```

## Background

With ``M = (L + R) / 2`` (mid) and ``S = (L - R) / 2`` (side), a stereo signal is ``(M + S, M - S)``. The mid branch carries ``(M, M)``, the side branch ``(S, -S)``. Sent to the same channel, they add up to the original ``(L, R)``. Anything you do to the side branch changes only the width and the space of the sound, not the center (vocals, kick, bass).

## Code

```ts
import { AudioClip, DspPipeline, LowPassFilter, StereoMono, loadAudioSource } from "@fluex/fluexgl-dsp";

async function start() {

    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    if (!await pipeline.initializeDpsPipeline()) return;

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    await audioDevice.context.resume();

    const master = audioDevice.getMasterChannel();

    // 1. The source. It is NOT sent to the master: only the branches should be heard.
    const music = audioDevice.createChannel("Music");
    const surround = audioDevice.createChannel("Surround");

    // 2. Split. Both branches get a StereoMono effect, and "music" is sent to both.
    const [mid, side] = StereoMono.split(music, "mid-side");

    // 3. Process the side into a diffuse rear: a Haas delay on one side, darker and louder.
    const sideRouting = side.effects[0] as StereoMono;

    sideRouting.setDelayRight(18);
    side.addEffect(new LowPassFilter({ cutoff: 7000 }));
    side.volume(1.4);

    // 4. Merge: both branches are sent to the same channel.
    mid.send(surround);
    side.send(surround);
    surround.send(master);

    // 5. Play some music.
    const data = await loadAudioSource("/audio/song.ogg");

    if (!data) return;

    const clip = new AudioClip(data);

    clip.send(music);
    clip.setLoop(true);
    clip.play();

    // 6. Bypass toggle: route "music" straight to the master, or through the split.
    document.querySelector("#surround")!.addEventListener("click", () => {

        if (music.isSentTo(mid)) {
            music.unsend(mid);
            music.unsend(side);
            music.send(master);
        } else {
            music.unsend(master);
            music.send(mid);
            music.send(side);
        }
    });
}

document.addEventListener("click", start, { once: true });
```

## Variations

### Left and right processed separately
```ts
const [left, right] = StereoMono.split(music, "left-right");

right.addEffect(new LowPassFilter({ cutoff: 2000 })); // only the right side gets darker

left.send(surround);
right.send(surround);
```

### A microphone (mono) through the surround
A mono signal has no side (``L = R``, so ``S = 0``), so the side branch stays silent. Create a stereo difference first with a short delay on one side:

```ts
const microphone = await audioDevice.createInputChannel(null, "Microphone");

microphone.addEffect(new StereoMono({ delayRightMs: 12 }));
microphone.send(music); // or split the microphone channel itself
```

### A mono button on the master
```ts
const monoCheck = new StereoMono();
master.attachEffect(monoCheck);

monoCheck.setMode("mono");   // check the mix in mono
monoCheck.setMode("stereo"); // back to normal
```

## Notes
- Do not send the source channel to the master as well, otherwise the original signal is heard on top of the processed branches. ``StereoMono.split()`` does not change existing sends.
- Keep the side delay below about 30 ms. Longer delays are heard as a separate echo instead of as width.
- Check the result in mono (``"mono"`` mode on the master): a delayed side partly cancels itself in mono, which is normal for widening effects, but should not make the mix fall apart.
- ``StereoMono`` uses native Web Audio nodes, so it works without WebAssembly. ``LowPassFilter`` needs the initialized pipeline.
