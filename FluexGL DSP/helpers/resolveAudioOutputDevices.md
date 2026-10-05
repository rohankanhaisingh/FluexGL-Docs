# resolveAudioOutputDevices

Resolves and returns a list of available audio output devices on the system.

```ts
async function resolveAudioOutputDevices(): Promise<AudioDevice[]>;
```

- - -

> **Deprecated.** This function creates an `AudioContext` (and a master channel) for every output device. Use [`listAudioOutputDevices()`](./listAudioOutputDevices.md) to list the devices, and [`AudioDevice.setOutputDevice()`](../classes/AudioDevice.md) to switch the output device of a single audio device at runtime. The function still works and is kept for backwards compatibility.

## About
The `resolveAudioOutputDevices()` function queries the browser for all available media devices and filters the result to return only audio output devices.

Each resolved device is wrapped in an `AudioDevice` abstraction, providing a consistent interface for interacting with output hardware such as speakers, headphones, or virtual audio devices. Each `AudioDevice` plays on its own output device (in browsers that support `AudioContext.setSinkId()`).

This function is commonly used to populate device selectors or to allow users to choose their preferred audio output destination.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
This function does not accept any parameters.

## Returns (promised)

- `AudioDevice[]` – An array of resolved audio output devices.  
  The array may be empty if no audio output devices are available or accessible.

## Error and warnings

### DOMException (enumerateDevices)
If the browser blocks access to media device enumeration due to missing permissions or insecure context, `enumerateDevices()` may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
