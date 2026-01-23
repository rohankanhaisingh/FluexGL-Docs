# ResolveDefaultAudioInputDevice

Resolves and initializes the system's default audio input device for DSP processing.

```ts
async function ResolveDefaultAudioInputDevice(
    init: DspPipelineInitializationState
): Promise<AudioDevice | null>;
```

- - -

## About
The `ResolveDefaultAudioInputDevice()` function attempts to locate the system's default audio input device and prepare it for DSP processing.

The function:
- Enumerates available media devices
- Identifies the default audio input device
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

This warning is emitted when the system reports no default audio input device.

### DOMException (enumerateDevices / AudioWorklet)
Browser-level exceptions may be thrown if media device enumeration fails or if the AudioWorklet cannot be attached to the resolved device.
