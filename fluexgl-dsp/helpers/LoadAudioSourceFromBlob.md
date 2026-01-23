# LoadAudioSourceFromBlob

A lightweight helper function within FluexGL DSP for decoding audio data directly from a Blob.

```ts
async function LoadAudioSourceFromBlob(blob: Blob): Promise<AudioSourceData | null>;
```

- - -

## About
The `LoadAudioSourceFromBlob()` function decodes audio data from a provided `Blob` object into an `AudioBuffer` using the Web Audio API.

This function is designed for situations where audio data is already available in memory rather than referenced by a file path or URL. Common use cases include file uploads, drag-and-drop interactions, or audio data retrieved via browser APIs such as `fetch()`.

Internally, a temporary `AudioContext` is created to perform the decoding step.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `blob`: `Blob` – A Blob containing raw audio data to be decoded.

## Returns (promised)

- `AudioSourceData` – An object containing the decoded audio data and metadata, including:
  - A unique identifier
  - A creation timestamp
  - The decoded `AudioBuffer`
  - The original `ArrayBuffer`
- `null` – Reserved for future error handling (currently not returned).

## Error and warnings

### DOMException (decodeAudioData)
If the provided `Blob` does not contain valid or supported audio data, the internal `decodeAudioData()` call may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
