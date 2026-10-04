# Example 02: Effect chains

Builds an effect chain on a channel, changes effect parameters at runtime, reorders and removes effects, and puts a limiter on the master channel.

## What it shows
- AudioWorklet effects ([``LowPassFilter``](../effects/LowPassFilter.md), [``Reverb``](../effects/Reverb.md)), which require the DSP pipeline to be initialized.
- Native effects ([``Compressor``](../effects/Compressor.md), [``Limiter``](../effects/Limiter.md)), which work without WebAssembly.
- Mixing both kinds in one chain.

## Routing

```
AudioClip -> Channel: [LowPassFilter -> Compressor -> Reverb] -> panner -> gain -> Master: [Limiter] -> speakers
```

## Code

```ts
import { AudioClip, Compressor, Limiter, LowPassFilter, Reverb } from "@fluex/fluexgl-dsp";

// Assumes `audioDevice` and `data` from Example 01.
const master = audioDevice.getMasterChannel();
const voice = audioDevice.createChannel("Voice");

voice.send(master);

const lowpass = new LowPassFilter({ cutoff: 4000 });
const compressor = new Compressor({ threshold: -20, ratio: 4, attack: 0.005, release: 0.2, makeupGain: 3 });
const reverb = new Reverb({ mix: 0.25, roomSize: 0.5 });

// Effects are processed in the order they are added. addEffect returns the channel, so calls can be chained.
voice
    .addEffect(lowpass)
    .addEffect(compressor)
    .addEffect(reverb);

// A safety limiter at the end of the master chain.
master.attachEffect(new Limiter({ ceiling: -1 }));

const clip = new AudioClip(data);
clip.send(voice);
clip.play();

// Change parameters at runtime.
lowpass.setCutoff(800);       // muffled, e.g. when the player is underwater
compressor.setThreshold(-30);
reverb.setMix(0.4);

// Read the gain reduction of the compressor, e.g. for a meter.
setInterval(() => console.log("reduction", compressor.reduction.toFixed(1), "dB"), 250);

// Reorder and remove.
voice.moveEffectToIndex(reverb, "start");
voice.removeEffect(lowpass);
```

## Notes
- AudioWorklet effects (most effects) can only be added after ``await pipeline.initializeDpsPipeline()`` and on the ``AudioDevice`` returned by ``resolveDefaultAudioOutputDevice()``, because the worklet is loaded on that device's ``AudioContext``. Check [``hasInitializedWasm``](../helpers/hasInitializedWasm.md) if unsure.
- ``Compressor`` and ``Limiter`` use the native ``DynamicsCompressorNode`` and work everywhere.
- ``LowPassFilter`` and ``Chorus`` require an options object: use ``new LowPassFilter({})``, not ``new LowPassFilter()``.
- Setters return ``false`` when the effect is not attached yet. Native effects store the value and apply it on attachment.
- To write your own effect from native Web Audio nodes, see the example at the bottom of [``Effector``](../classes/Effector.md).
