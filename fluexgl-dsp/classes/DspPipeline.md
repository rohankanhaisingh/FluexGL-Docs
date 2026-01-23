# Class ``DspPipeline``

Represents the digital signal processing (DSP) pipeline responsible for initializing WebAssembly modules, AudioWorklet processors, and resolving audio output devices.

## Example

```ts
import { DspPipeline } from "@fluex/fluexgl-dsp";

const pipeline = new DspPipeline({
  pathToWasm: "/dsp/dsp.wasm",
  pathToWorklet: "/dsp/processor.worklet.js",
  options: { /* DSP global options */ }
});

await pipeline.Init();
const device = await pipeline.ResolveDefaultAudioOutputDevice();
```

- - -

## Constructor
Constructs a new DSP pipeline using the provided initialization options (WASM path, worklet path, and global DSP options).

```ts
new DspPipeline({ pathToWasm, pathToWorklet, options }: DspPipelineInitializationOptions): DspPipeline;
```

### Arguments
- ``options``: ``DspPipelineInitializationOptions`` - Initialization object containing ``pathToWasm``, ``pathToWorklet``, and optional DSP configuration.

- - -

## Properties

### ``pathToWasm: string | null``
### ``pathToWorklet: string | null``
### ``id: string``
### ``hasInitialized: boolean``

- - -

## Methods

### ``Init(): Promise<boolean>``

#### Arguments
No arguments

#### Returns
- ``Promise<boolean>``

### ``InitializeDpsPipeline(): Promise<boolean>``

#### Arguments
No arguments

#### Returns
- ``Promise<boolean>``

### ``ResolveDefaultAudioOutputDevice(): Promise<AudioDevice | null>``

#### Arguments
No arguments

#### Returns
- ``Promise<AudioDevice | null>`` - The default [``AudioDevice``](./AudioDevice.md) instance, or ``null`` when no default output device is found or initialization is missing.

### ``TellMeWhatTheFuckThisWholeLibraryActuallyDoes(): void``


#### Arguments
No arguments

#### Returns
- ``void``

## Events

This class does not emit custom events.

- - -

## Getters and setters

This class does not define public getters or setters.

- - -

## Examples

### Example 1: initialize and resolve the default output device
```ts
const pipeline = new DspPipeline({ pathToWasm, pathToWorklet, options: {} });
const ok = await pipeline.InitializeDpsPipeline();
if (!ok) throw new Error("DSP pipeline init failed");

const device = await pipeline.ResolveDefaultAudioOutputDevice();
if (!device) throw new Error("No default audio output device");
```

### Example 2: quick init via Init()
```ts
const pipeline = new DspPipeline({ pathToWasm, pathToWorklet, options: {} });
await pipeline.Init();
```
