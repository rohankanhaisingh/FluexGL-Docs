# CreateAudioWorkletNode

Creates and initializes an AudioWorkletNode bound to the FluexGL DSP WebAssembly module.

```ts
function CreateAudioWorkletNode<T = any>(
    context: AudioContext,
    name: AudioWorkletProcessorNames | string,
    data: T
): AudioWorkletNode | null;
```

- - -

## About
The `CreateAudioWorkletNode()` function constructs an `AudioWorkletNode` configured for FluexGL DSP processing.

The function:
- Verifies that the DSP WebAssembly module has been compiled and is available
- Injects DSP parameters and runtime metadata (such as `sampleRate`)
- Binds the compiled WebAssembly module to the AudioWorklet processor
- Applies a standardized channel and I/O configuration

This utility is the primary factory for creating DSP-enabled AudioWorklet nodes and should be used instead of instantiating `AudioWorkletNode` directly.

## Parameters
- `context`: `AudioContext` – The AudioContext on which the AudioWorkletNode will be created.
- `name`: `AudioWorkletProcessorNames | string` – The registered AudioWorklet processor name.
- `data`: `T` – Initial parameter data passed to the processor, merged with runtime metadata.

## Returns

- `AudioWorkletNode` – A fully configured AudioWorklet node ready for DSP processing.
- `null` – Returned if the WebAssembly module is unavailable or node creation fails.

## Error and warnings

### `ERROR:FLUEXGL-DSP@0004`
The WebAssembly module has not been compiled or loaded correctly.

This error is emitted if `CreateAudioWorkletNode()` is called before the DSP pipeline has been initialized.

### `ERROR:FLUEXGL-DSP@0005`
Failed to create AudioWorklet node.

Possible causes include:
- The AudioWorklet module was not loaded on the provided AudioContext
- An invalid processor name was provided
- Browser-level AudioWorklet errors

Additional diagnostic information is logged via the internal debug logger.
