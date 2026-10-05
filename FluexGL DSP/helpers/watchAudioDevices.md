# watchAudioDevices

Calls a callback every time an audio input or output device is connected or disconnected.

```ts
function watchAudioDevices(callback: (event: AudioDeviceListChangedEvent) => void): () => void;
```

- - -

## About
The `watchAudioDevices()` function listens for the `devicechange` event of `navigator.mediaDevices`, enumerates the devices again, and passes the current audio inputs and outputs to the callback. Useful to keep a device selection menu up to date.

The callback is only called on changes, not right away. Call [`listAudioInputDevices()`](./listAudioInputDevices.md) and [`listAudioOutputDevices()`](./listAudioOutputDevices.md) once for the initial lists.

An [`AudioDevice`](../classes/AudioDevice.md) also fires a `devices-changed` event with the same payload. Use `watchAudioDevices()` when you do not have (or do not want to depend on) an audio device.

You do not need this function to handle an unplugged device: [`AudioDevice`](../classes/AudioDevice.md) and [`InputChannel`](../classes/InputChannel.md) fall back to the default device by themselves.

## Parameters
- `callback`: `(event: AudioDeviceListChangedEvent) => void` – Called with the current device lists. See [`AudioDeviceListChangedEvent`](../interfaces/AudioDeviceListChangedEvent.md).

## Returns

- `() => void` – A function that stops watching.

## Example

```ts
import { watchAudioDevices } from "@fluex/fluexgl-dsp";

const stopWatching = watchAudioDevices(({ inputs, outputs }) => {
    renderDeviceMenu(inputs, outputs);
});

// Later:
stopWatching();
```

## Error and warnings

No custom FluexGL-DSP error codes are currently emitted by this function.
