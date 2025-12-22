# DspPipelineInitializationOptions

Configuration options for initializing the DSP pipeline in FluexGL DSP.

```ts
interface DspPipelineInitializationOptions {
    pathToWasm: string;
    pathToWorklet: string;
    options?: Partial<DspOptions>;
}
```

## About
Specifies options used when initializing the DSP pipeline.

## Properties
- `pathToWasm`: `string` - Path to the WebAssembly file.
- `pathToWorklet`: `string` - Path to the worklet file.
- `options?`: `Partial<DspOptions>` - Optional DSP configuration overrides.

