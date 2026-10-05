# AudioDeviceLostEvent

Payload of the events fired when a device has been disconnected.

```ts
interface AudioDeviceLostEvent {
    lost: MediaDeviceInfo | null;
    timestamp: number;
}
```

## About
Used by the ``output-device-lost`` event of [``AudioDevice``](../classes/AudioDevice.md) and the ``input-device-lost`` event of [``InputChannel``](../classes/InputChannel.md).

## Properties
- `lost`: `MediaDeviceInfo | null` - The device that has been disconnected. ``null`` when unknown, or when an external stream ended.
- `timestamp`: `number` - ``Date.now()`` at the moment the loss was detected.
