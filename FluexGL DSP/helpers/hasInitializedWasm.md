# hasInitializedWasm

A mutable flag indicating whether the FluexGL DSP WebAssembly module has been compiled.

```ts
let hasInitializedWasm: boolean;
```

- - -

## About
``hasInitializedWasm`` is a top-level, mutable ``boolean`` binding exported from the package. It is initialized to ``false`` and set to ``true`` as soon as the WebAssembly module has been compiled successfully, which happens during [``DspPipeline.initializeDpsPipeline()``](../classes/DspPipeline.md).

Because it is an ES module live binding, an imported ``hasInitializedWasm`` always reflects the current value.

Effects that run on an AudioWorklet (such as [``Reverb``](../effects/Reverb.md) or [``LowPassFilter``](../effects/LowPassFilter.md)) can only be attached once this flag is ``true``. Native effects such as [``Compressor``](../effects/Compressor.md) and [``Limiter``](../effects/Limiter.md) do not depend on it.

The spatial renderers use this flag to attach their default reverb automatically once WebAssembly is ready.

## Example

```ts
import { hasInitializedWasm, Reverb } from "@fluex/fluexgl-dsp";

if (hasInitializedWasm)
    channel.addEffect(new Reverb({ mix: 0.3 }));
```

## Value
- `boolean` - ``true`` once the WebAssembly module has been compiled, otherwise ``false``.

## Error and warnings

This value does not emit any errors or warnings.
