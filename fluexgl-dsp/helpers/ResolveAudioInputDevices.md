# ResolveAudioInputDevices

Resolves and returns a list of available audio input devices on the system.

```ts
async function ResolveAudioInputDevices(): Promise<AudioDevice[]>;
```

- - -

## About
The `ResolveAudioInputDevices()` function queries the browser for all available media devices and filters the result to return only audio input devices.

Each resolved device is wrapped in an `AudioDevice` abstraction, providing a consistent interface for interacting with input hardware such as microphones, audio interfaces, or virtual input devices.

This function is typically used during DSP pipeline initialization or to allow users to select an input device for audio capture or processing.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
This function does not accept any parameters.

## Returns (promised)

- `AudioDevice[]` – An array of resolved audio input devices.  
  The array may be empty if no audio input devices are available or accessible.

## Error and warnings

### DOMException (enumerateDevices)
If the browser blocks access to media device enumeration due to missing permissions or an insecure context, `enumerateDevices()` may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
