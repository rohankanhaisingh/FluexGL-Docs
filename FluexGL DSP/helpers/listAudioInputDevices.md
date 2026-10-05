# listAudioInputDevices

Returns the available audio input devices, such as microphones, audio interfaces and line-ins.

```ts
async function listAudioInputDevices(): Promise<MediaDeviceInfo[]>;
```

- - -

## About
The `listAudioInputDevices()` function queries the browser for all available media devices and returns only the audio input devices, as plain `MediaDeviceInfo` objects.

Use the result with [`AudioDevice.createInputChannel()`](../classes/AudioDevice.md) to open a device on a new [`InputChannel`](../classes/InputChannel.md), or with [`InputChannel.setInputDevice()`](../classes/InputChannel.md) to switch the device of an existing input channel. Unlike the deprecated [`resolveAudioInputDevices()`](./resolveAudioInputDevices.md), this function does not create an `AudioContext` for every device.

Device labels are only filled in after the user granted permission to access media devices, which [`DspPipeline.initializeDpsPipeline()`](../classes/DspPipeline.md) requests.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
This function does not accept any parameters.

## Returns (promised)

- `MediaDeviceInfo[]` – All devices of kind `"audioinput"`.  
  The array may be empty if no audio input devices are available or accessible.

## Example

```ts
import { listAudioInputDevices } from "@fluex/fluexgl-dsp";

const inputs = await listAudioInputDevices();
const microphone = await audioDevice.createInputChannel(inputs[0], "Microphone");
```

## Error and warnings

### DOMException (enumerateDevices)
If the browser blocks access to media device enumeration due to missing permissions or an insecure context, `enumerateDevices()` may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
