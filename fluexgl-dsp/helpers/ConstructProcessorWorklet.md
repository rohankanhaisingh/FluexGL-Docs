# ConstructProcessorWorklet

Constructs a Blob URL for an AudioWorklet processor from provided JavaScript source code.

```ts
function ConstructProcessorWorklet(code: string): string;
```

- - -

## About
The `ConstructProcessorWorklet()` function creates a Blob from a provided JavaScript source string and converts it into an object URL that can be used to register an `AudioWorkletProcessor`.

This function is primarily used during DSP pipeline initialization to dynamically construct AudioWorklet processor files at runtime, allowing flexible loading and bundling strategies.

The resulting Blob URL can be passed directly to `audioContext.audioWorklet.addModule()` or similar initialization utilities.

## Parameters
- `code`: `string` – JavaScript source code defining an AudioWorklet processor.

## Returns

- `string` – A Blob URL referencing the constructed AudioWorklet processor source.

## Error and warnings

No custom FluexGL-DSP errors or warnings are emitted by this function.

Browser-level errors may occur if the generated code is invalid or if the Blob URL cannot be consumed by the AudioWorklet API.
