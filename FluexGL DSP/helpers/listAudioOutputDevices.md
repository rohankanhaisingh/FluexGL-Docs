# listAudioOutputDevices

Returns the available audio output devices, such as speakers and headphones.

```ts
async function listAudioOutputDevices(): Promise<MediaDeviceInfo[]>;
```

- - -

## About
The `listAudioOutputDevices()` function queries the browser for all available media devices and returns only the audio output devices, as plain `MediaDeviceInfo` objects.

Use the result with [`AudioDevice.setOutputDevice()`](../classes/AudioDevice.md) to switch the output device of an existing audio device, without recreating the `AudioContext` and without reloading the page. Unlike the deprecated [`resolveAudioOutputDevices()`](./resolveAudioOutputDevices.md), this function does not create an `AudioContext` for every device.

Device labels are only filled in after the user granted permission to access media devices, which [`DspPipeline.initializeDpsPipeline()`](../classes/DspPipeline.md) requests. Chromium based browsers also list the pseudo devices `"default"` and `"communications"`.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
This function does not accept any parameters.

## Returns (promised)

- `MediaDeviceInfo[]` – All devices of kind `"audiooutput"`.  
  The array may be empty, for example in browsers that do not expose output devices.

## Example

```ts
import { AudioDevice, listAudioOutputDevices } from "@fluex/fluexgl-dsp";

const outputs = await listAudioOutputDevices();

if (AudioDevice.supportsOutputDeviceSelection)
    await audioDevice.setOutputDevice(outputs[1]);
```

## Error and warnings

### DOMException (enumerateDevices)
If the browser blocks access to media device enumeration due to missing permissions or an insecure context, `enumerateDevices()` may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
