# hasInitializedWasm

A mutable flag indicating whether the FluexGL DSP WebAssembly module has been compiled.

```ts
let hasInitializedWasm: boolean;
```

- - -

## About
``hasInitializedWasm`` is a top-level, mutable ``boolean`` binding exported from the package. It is initialized to ``false``.

<!-- TODO: verify -- in the current source (utilities/web-assembly.ts) this flag is declared but never actually reassigned anywhere (the module instead tracks readiness via the separate `compiledWebAssemblyModule` variable used internally by createAudioWorkletNode). It is documented here as declared, but it will read `false` even after a successful DSP pipeline initialization; treat it as not yet wired up rather than a reliable "is WASM ready" check. -->

## Value
- `boolean` – Always `false` as currently implemented.

## Error and warnings

This value does not emit any errors or warnings.
