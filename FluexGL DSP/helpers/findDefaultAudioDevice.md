# findDefaultAudioDevice

Finds the system's default audio input or output device.

```ts
async function findDefaultAudioDevice(kind: "audioinput" | "audiooutput"): Promise<MediaDeviceInfo | null>;
```

- - -

## About
The `findDefaultAudioDevice()` function returns the `MediaDeviceInfo` of the system's default device of the given kind.

Chromium based browsers list a pseudo device with the id `"default"`, which is returned when present. Browsers without such an entry (such as Firefox) list the default device first, so the first device of that kind is used as a fallback.

[`resolveDefaultAudioOutputDevice()`](./resolveDefaultAudioOutputDevice.md) and [`DspPipeline.resolveDefaultAudioOutputDevice()`](../classes/DspPipeline.md) use this function internally.

Note: This function is asynchronous and must be called within an asynchronous scope.

## Parameters
- `kind`: `"audioinput" | "audiooutput"` – The kind of device to look for.

## Returns (promised)

- `MediaDeviceInfo` – The default device of the given kind.
- `null` – Returned if there is no device of the given kind.

## Example

```ts
import { findDefaultAudioDevice } from "@fluex/fluexgl-dsp";

const defaultMicrophone = await findDefaultAudioDevice("audioinput");

console.log(defaultMicrophone?.label);
```

## Error and warnings

### DOMException (enumerateDevices)
If the browser blocks access to media device enumeration due to missing permissions or an insecure context, `enumerateDevices()` may throw a `DOMException`.

No custom FluexGL-DSP error codes are currently emitted by this function.
