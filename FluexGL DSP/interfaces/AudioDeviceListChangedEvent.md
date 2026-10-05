# AudioDeviceListChangedEvent

The current audio devices, after a device was connected or disconnected.

```ts
interface AudioDeviceListChangedEvent {
    inputs: MediaDeviceInfo[];
    outputs: MediaDeviceInfo[];
    timestamp: number;
}
```

## About
Passed to the callback of [``watchAudioDevices()``](../helpers/watchAudioDevices.md) and to the ``devices-changed`` event of [``AudioDevice``](../classes/AudioDevice.md).

## Properties
- `inputs`: `MediaDeviceInfo[]` - All devices of kind ``"audioinput"``, the same as [``listAudioInputDevices()``](../helpers/listAudioInputDevices.md).
- `outputs`: `MediaDeviceInfo[]` - All devices of kind ``"audiooutput"``, the same as [``listAudioOutputDevices()``](../helpers/listAudioOutputDevices.md).
- `timestamp`: `number` - ``Date.now()`` at the moment the devices were enumerated.
