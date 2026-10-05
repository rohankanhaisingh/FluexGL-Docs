# resolveAudioInputDevices

Resolves and returns a list of available audio input devices on the system.

```ts
async function resolveAudioInputDevices(): Promise<AudioDevice[]>;
```

- - -

> **Deprecated.** An input device has no `AudioContext` of its own, but this function creates an `AudioContext` (and a master channel) for every input device. Use [`listAudioInputDevices()`](./listAudioInputDevices.md) to list the devices, and [`AudioDevice.createInputChannel()`](../classes/AudioDevice.md) to open one. The function still works and is kept for backwards compatibility.

## About
The `resolveAudioInputDevices()` function queries the browser for all available media devices and filters the result to return only audio input devices.

Each resolved device is wrapped in an `AudioDevice` abstraction, providing a consistent interface for interacting with input hardware such as microphones, audio interfaces, or virtual input devices.

The returned `AudioDevice` instances do not capture audio. To actually record from an input device, use an [`InputChannel`](../classes/InputChannel.md).

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
