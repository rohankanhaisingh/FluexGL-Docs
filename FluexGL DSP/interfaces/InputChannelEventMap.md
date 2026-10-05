# InputChannelEventMap

Maps InputChannel events to handlers.

```ts
interface InputChannelEventMap {
    "input-device-changed": (event: AudioDeviceChangedEvent) => void;
    "input-device-lost": (event: AudioDeviceLostEvent) => void;
}
```

## About
Used by [``InputChannel.addEventListener()``](../classes/InputChannel.md).

## Properties
- `input-device-changed`: `(event: AudioDeviceChangedEvent) => void` - Fired when a device was opened with ``setInputDevice()``, or a stream was connected with ``setMediaStream()`` (``current`` is then ``null``). See [``AudioDeviceChangedEvent``](./AudioDeviceChangedEvent.md).
- `input-device-lost`: `(event: AudioDeviceLostEvent) => void` - Fired when the device was unplugged, permission was revoked, or an external stream ended. For a local device with ``fallbackToDefault`` enabled, the default input device is opened afterwards. See [``AudioDeviceLostEvent``](./AudioDeviceLostEvent.md).
