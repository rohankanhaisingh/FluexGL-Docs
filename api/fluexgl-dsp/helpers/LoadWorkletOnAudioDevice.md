# LoadWorkletOnAudioDevice

Loads and registers AudioWorklet processor modules on a specific audio device.

```ts
async function LoadWorkletOnAudioDevice(
    audioDevice: AudioDevice,
    workletBlobUrl: string
): Promise<boolean>;
```

- - -

## About
The `LoadWorkletOnAudioDevice()` function registers an AudioWorklet processor module on the `AudioContext` associated with a given `AudioDevice`.

This function:
- Attaches DSP worklet modules to the device's master audio context
- Measures and logs load performance
- Finalizes AudioWorklet availability for downstream DSP processing

It is typically called after resolving an audio device and constructing the AudioWorklet processor Blob URL.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `audioDevice`: `AudioDevice` – The target audio device whose `AudioContext` will receive the AudioWorklet module.
- `workletBlobUrl`: `string` – A Blob URL referencing the AudioWorklet processor source.

## Returns (promised)

- `boolean` – Returns `true` once the AudioWorklet module has been successfully loaded and registered.

## Error and warnings

### DOMException (audioWorklet.addModule)
Browser-level exceptions may be thrown if:
- The AudioWorklet source code is invalid
- The Blob URL cannot be resolved
- AudioWorklet is not supported in the current execution context

No custom FluexGL-DSP error codes are currently emitted by this function.
