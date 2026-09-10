# initializeDspPipeline

Initializes the FluexGL DSP pipeline by preparing audio permissions, loading WebAssembly modules, and constructing the AudioWorklet processor.

```ts
async function initializeDspPipeline(
    options: DspPipelineInitializationOptions
): Promise<DspPipelineInitializationState | null>;
```

- - -

## About
The `initializeDspPipeline()` function performs all required setup steps to prepare the FluexGL DSP runtime environment.

Note this is a standalone module-level function, distinct from (and not called by) the [``DspPipeline``](../classes/DspPipeline.md) class's own ``initializeDpsPipeline()`` method — both perform the same steps independently.

This includes:
- Verifying permission to access audio input devices
- Loading the DSP WebAssembly module
- Fetching and constructing the AudioWorklet processor
- Measuring and reporting initialization performance

The function is intended to be called once during application startup before any DSP processing is performed.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `options`: `DspPipelineInitializationOptions` – Configuration object containing:
  - `pathToWasm`: Path or URL to the DSP WebAssembly module
  - `pathToWorklet`: Path or URL to the AudioWorklet processor source
  - `options?`: `Partial<DspOptions>` – Optional global DSP configuration overrides. Note: unlike the ``DspPipeline`` class constructor, this standalone function does not apply these overrides to the global ``DSP`` options object itself.

## Returns (promised)

- `DspPipelineInitializationState` – An object containing:
  - `success`: Indicates whether initialization completed successfully
  - `workletBlobUrl`: A Blob URL referencing the constructed AudioWorklet processor
- `null` – Reserved for future failure handling (currently not returned).

## Error and warnings

### `ERROR:FLUEXGL-DSP@0001`
Permission to access media devices was not granted.  
`ErrorCodes.NO_CONTEXT_PERMISSION`

This error is emitted if the browser denies access to audio input devices required to initialize the DSP pipeline.

---

Initialization performance metrics are logged via the internal debug logger.
