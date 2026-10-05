# resolveDefaultAudioInputDevice

Resolves and initializes the system's default audio input device for DSP processing.

```ts
async function resolveDefaultAudioInputDevice(
    init: DspPipelineInitializationState
): Promise<AudioDevice | null>;
```

- - -

> **Deprecated.** An input device has no `AudioContext` of its own, and the returned `AudioDevice` does not capture audio. Use [`findDefaultAudioDevice("audioinput")`](./findDefaultAudioDevice.md) to find the default input device, or call [`AudioDevice.createInputChannel()`](../classes/AudioDevice.md) without a device to open it on an [`InputChannel`](../classes/InputChannel.md). The function still works and is kept for backwards compatibility.

## About
The `resolveDefaultAudioInputDevice()` function attempts to locate the system's default audio input device and prepare it for DSP processing.

The function:
- Enumerates available media devices
- Identifies the default audio input device (the `"default"` entry, or the first input device in browsers without one, see [`findDefaultAudioDevice()`](./findDefaultAudioDevice.md))
- Attaches the DSP AudioWorklet processor to the resolved device

This function is typically called after successful DSP pipeline initialization and is required before capturing or processing live audio input.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `init`: `DspPipelineInitializationState` – The DSP pipeline initialization state containing:
  - `workletBlobUrl`: Blob URL pointing to the constructed AudioWorklet processor

## Returns (promised)

- `AudioDevice` – The resolved and initialized default audio input device.
- `null` – Returned if no default audio input device could be found or initialized.

## Error and warnings

### `WARNING:FLUEXGL-DSP@0001`
No default audio input device was found.  
`WarningCodes.NO_DEFAULT_AUDIO_DEVICE_FOUND`

This warning is emitted when the system reports no audio input device at all. The function then returns `null`.

### DOMException (enumerateDevices / AudioWorklet)
Browser-level exceptions may be thrown if media device enumeration fails or if the AudioWorklet cannot be attached to the resolved device.
