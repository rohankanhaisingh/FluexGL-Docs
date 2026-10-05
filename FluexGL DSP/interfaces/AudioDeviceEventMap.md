# AudioDeviceEventMap

Maps AudioDevice events to handlers.

```ts
interface AudioDeviceEventMap {
    "output-device-changed": (event: AudioDeviceChangedEvent) => void;
    "output-device-lost": (event: AudioDeviceLostEvent) => void;
    "devices-changed": (event: AudioDeviceListChangedEvent) => void;
}
```

## About
Used by [``AudioDevice.addEventListener()``](../classes/AudioDevice.md).

## Properties
- `output-device-changed`: `(event: AudioDeviceChangedEvent) => void` - Fired when ``setOutputDevice()`` switched the output device. See [``AudioDeviceChangedEvent``](./AudioDeviceChangedEvent.md).
- `output-device-lost`: `(event: AudioDeviceLostEvent) => void` - Fired when the selected output device is disconnected. The audio device then falls back to the default output device. See [``AudioDeviceLostEvent``](./AudioDeviceLostEvent.md).
- `devices-changed`: `(event: AudioDeviceListChangedEvent) => void` - Fired when an audio input or output device is connected or disconnected. See [``AudioDeviceListChangedEvent``](./AudioDeviceListChangedEvent.md).
