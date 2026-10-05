# AudioDeviceChangedEvent

Payload of the events fired when an input or output device has been switched.

```ts
interface AudioDeviceChangedEvent {
    previous: MediaDeviceInfo | null;
    current: MediaDeviceInfo | null;
    timestamp: number;
}
```

## About
Used by the ``output-device-changed`` event of [``AudioDevice``](../classes/AudioDevice.md) and the ``input-device-changed`` event of [``InputChannel``](../classes/InputChannel.md).

## Properties
- `previous`: `MediaDeviceInfo | null` - The device that was used before the switch. ``null`` when unknown, or when nothing was opened before.
- `current`: `MediaDeviceInfo | null` - The device that is used now. ``null`` when the browser does not expose the device info, or when an external stream is used (``InputChannel.setMediaStream()``).
- `timestamp`: `number` - ``Date.now()`` at the moment of the switch.
