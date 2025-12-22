# Getting started with FluexGL DSP

This guide helps you set up FluexGL DSP for your project. The setup is straightforward but requires a few specific settings.

---

## Installation

To use FluexGL DSP, install the following components.

### 1. NPM package
From your project directory, install FluexGL DSP with NPM. Always use the latest version to stay up to date.

```
$ npm install @fluex/fluexgl-dsp
```

### 2. FluexGL DSP WebAssembly
FluexGL DSP uses WebAssembly for high-performance audio processing. The WebAssembly build lives in a separate repository and is not bundled with the NPM package (``@fluex/fluexgl-dsp``), so you must provide it manually.

#### 2.1 Go to the GitHub repository
Go to the GitHub releases page to find the latest version: [https://github.com/rohankanhaisingh/FluexGL-DSP-WebAssembly/releases](https://github.com/rohankanhaisingh/FluexGL-DSP-WebAssembly/releases).

#### 2.2 Download the latest release
Download the latest release (a ``.zip`` file) containing the required files. The filename is usually ``fluexgl-dsp-wasm-release-x.x.x.zip``.

**Note:** Install the WebAssembly version that matches the NPM package version. Mixing versions can break features.

#### 2.3 Extract and prepare
Extract the ``.zip`` file into a new folder. The folder contains the following files:

- ``fluexgl-dsp-processor.worklet`` (**important**)
- ``fluexgl-dsp.wasm.old.js``
- ``fluexgl-dsp-wasm.js``
- ``fluexgl-dsp-wasm_bg.wasm`` (**important**)
- ``package.json``
- ``.gitignore``
- ``fluexgl-dsp-wasm.d.ts``
- ``fluexgl-dsp-wasm_bg.wasm.d.ts``

Move the important files into a folder in your project and ensure your server can serve them. A common path is ``/public/data/fluexgl-dsp/``. FluexGL DSP uses the built-in ``fetch`` method to load both the processor worklet and the WebAssembly file.

### 3. Initializing the DSP pipeline
After installing the NPM package and the WebAssembly release, you are ready to initialize the DSP pipeline in your client code.

#### 3.1 Set up the initialization pipeline
First, set up the initialization pipeline with the ``DspPipeline`` class. Wrap the code in an asynchronous scope.

```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "path_to_wasm.wasm",
        pathToWorklet: "path_to_worklet.worklet"
    });
})();
```

#### 3.2 Provide the pipeline with the worklet and WASM paths
This class requires an options object. The required options are ``pathToWasm`` and ``pathToWorklet``.

Enter the paths to the ``.wasm`` and ``.worklet`` files that the ``DspPipeline`` can reach. If the files are not accessible, the console shows a detailed error.

Correct example usage:
```
(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp/fluexgl-dsp-processor.worklet"
    });
})();
```

#### 3.3 Initialize the pipeline
After providing the required data, initialize the pipeline using ``InitializeDpsPipeline()`` or the shorthand ``Init()`` method.

```ts
(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp/fluexgl-dsp-processor.worklet"
    });

    await pipeline.InitializeDpsPipeline();
})();
```

Check the console to follow the initialization process. Each step is logged.

Great! You are now ready to use FluexGL DSP.

---

## More options
The ``DspPipeline`` class has additional configuration options. For example, you can control logging or allow FluexGL DSP to override the maximum audio buffer nodes.

### Disabling console logs
When deploying your application, you can disable logs so the console stays clean for users. Set this option manually:

```ts
(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp/fluexgl-dsp-processor.worklet",
        options: {
            debugger: {
                breakOnError: true,
                showInfo: false,
                showErrors: false,
                showWarnings: false
            }
        }
    });

    await pipeline.InitializeDpsPipeline();
})();
```

### Override the maximum number of audio buffer nodes
This is useful in scenarios like video games where you do not want to create a new ``AudioClip`` every time a sound plays. Instead, reuse audio buffer nodes within the same clip to keep the audio thread fast and optimized.

Enable the option:
```ts
(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.InitializeDpsPipeline();
})();
```

In your ``AudioClip``, manually set the maximum number of buffer source nodes:
```ts
import { AudioClip } from "@fluex/fluexgl-dsp";

const myAudioClip = new AudioClip(sourceData);
myAudioClip.SetMaxAudioBufferSourceNodes(200);
```
