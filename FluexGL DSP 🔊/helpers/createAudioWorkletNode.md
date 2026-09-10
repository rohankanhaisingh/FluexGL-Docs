# createAudioWorkletNode

Creates and initializes an AudioWorkletNode bound to the FluexGL DSP WebAssembly module.

```ts
function createAudioWorkletNode<T = any>(
    context: AudioContext,
    name: AudioWorkletProcessorNames | string,
    data: T
): AudioWorkletNode;
```

> **Note:** this function is defined and exported from ``utilities/helpers.ts``, but it is **not** re-exported from the package's top-level ``index.ts``. It is not currently reachable as ``import { createAudioWorkletNode } from "@fluex/fluexgl-dsp"`` — it is used internally by every effect class's ``initializeOnAttachment()`` (e.g. [``Chorus``](../effects/Chorus.md), [``LowPassFilter``](../effects/LowPassFilter.md)). Documented here for completeness.

- - -

## About
The `createAudioWorkletNode()` function constructs an `AudioWorkletNode` configured for FluexGL DSP processing.

The function:
- Verifies that the DSP WebAssembly module has been compiled and is available
- Injects DSP parameters and runtime metadata (such as `sampleRate`)
- Binds the compiled WebAssembly module to the AudioWorklet processor
- Applies a standardized channel and I/O configuration (``numberOfInputs: 1``, ``numberOfOutputs: 1``, ``outputChannelCount: [2]``)

This utility is the primary factory for creating DSP-enabled AudioWorklet nodes and should be used instead of instantiating `AudioWorkletNode` directly.

## Parameters
- `context`: `AudioContext` – The AudioContext on which the AudioWorkletNode will be created.
- `name`: `AudioWorkletProcessorNames | string` – The registered AudioWorklet processor name.
- `data`: `T` – Initial parameter data passed to the processor as ``parameterData``, merged with ``sampleRate``.

## Returns

- `AudioWorkletNode` – A fully configured AudioWorklet node ready for DSP processing.

## Error and warnings

### Thrown ``Error``
If the DSP WebAssembly module has not been compiled yet (i.e. [``initializeDspPipeline()``](./initializeDspPipeline.md) / [``DspPipeline.initializeDpsPipeline()``](../classes/DspPipeline.md) has not completed successfully), this function throws a plain JavaScript ``Error`` with the message:

> "Coult not create audio worklet node. WebAssembly has not been compiled yet."

No custom FluexGL-DSP error codes (``ErrorCodes``) are currently emitted by this function — it does not use the internal debug logger either.
