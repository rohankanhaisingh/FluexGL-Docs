# InitializeDspPipeline

Initializes the FluexGL DSP pipeline by preparing audio permissions, loading WebAssembly modules, and constructing the AudioWorklet processor.

```ts
async function InitializeDspPipeline(
    options: DspPipelineInitializationOptions
): Promise<DspPipelineInitializationState | null>;
```

- - -

## About
The `InitializeDspPipeline()` function performs all required setup steps to prepare the FluexGL DSP runtime environment.

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
