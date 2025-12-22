# Multiple audio buffer sources on an AudioClip

## About
Every AudioClip has a maximum number of audio buffer sources and is set to 1 by default. However, in some cases you may want to create multiple audio buffer sources for a single AudioClip.

For example, in a game where the player is shooting a gun, you would not create a new AudioClip for every individual gunshot. Instead, you want the buffer to duplicate itself and play as often as the gun fires.

You can achieve this by allowing FluexGL DSP to override the maximum number of audio buffer source nodes and by configuring the maximum amount per AudioClip.

These settings help keep performance optimal and stable, without creating unnecessary instances or flooding memory with unused objects.

## Usage
When initializing the DspPipeline, you must provide the initialization option ``overrideMaxAudioBufferNodes``. For example:

```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

(async function() {

    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.Init();
})();
```

This allows FluexGL DSP to override the maximum number of audio buffer source nodes. You can now manually configure the maximum amount for each specific audio clip. For example:

```ts
import { AudioClip } from "@fluex/fluexgl-dsp";

const clip = new AudioClip(sourceData);
clip.SetMaxAudioBufferSourceNodes(100);
```

Now, every time you play the audio clip, the buffer source is duplicated and played immediately, instead of waiting for the first buffer source to finish before creating a new one.