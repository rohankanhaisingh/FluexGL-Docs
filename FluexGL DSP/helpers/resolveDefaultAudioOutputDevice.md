# resolveDefaultAudioOutputDevice

Resolves and initializes the system's default audio output device for DSP processing.

```ts
async function resolveDefaultAudioOutputDevice(
    init: DspPipelineInitializationState
): Promise<AudioDevice | null>;
```

- - -

## About
The `resolveDefaultAudioOutputDevice()` function attempts to locate the system's default audio output device and prepare it for DSP processing.

The function:
- Enumerates available media devices
- Identifies the default audio output device (the `"default"` entry, or the first output device in browsers without one, such as Firefox; see [`findDefaultAudioDevice()`](./findDefaultAudioDevice.md))
- Attaches the DSP AudioWorklet processor to the resolved device

This function is typically called after successful DSP pipeline initialization and serves as the final step before audio playback or processing begins.

The output device of the returned `AudioDevice` can be switched later with [`setOutputDevice()`](../classes/AudioDevice.md), so there is no need to resolve a new device when the user picks another output.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `init`: `DspPipelineInitializationState` – The DSP pipeline initialization state containing:
  - `workletBlobUrl`: Blob URL pointing to the constructed AudioWorklet processor

## Returns (promised)

- `AudioDevice` – The resolved and initialized default audio output device.
- `null` – Returned if no default audio output device could be found or initialized.

## Error and warnings

### `WARNING:FLUEXGL-DSP@0001`
No default audio output device was found.  
`WarningCodes.NO_DEFAULT_AUDIO_DEVICE_FOUND`

This warning is emitted when the system reports no audio output device at all. The function then returns `null`.

### DOMException (enumerateDevices / AudioWorklet)
Browser-level exceptions may be thrown if media device enumeration fails or if the AudioWorklet cannot be attached to the resolved device.
