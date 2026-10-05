# Class ``InputChannel``

A [``Channel``](./Channel.md) that receives its signal from an audio input device, such as a microphone or line-in, or from an existing ``MediaStream``, such as the voice of another player received over WebRTC.

Because it extends [``Channel``](./Channel.md), effects, volume, panning and sends work exactly the same as on a normal channel. The input device can be switched at any time with ``setInputDevice()``, without rebuilding the channel, its effects or its sends.

Extends [``Channel``](./Channel.md).

## Example

```ts
import { listAudioInputDevices, Compressor } from "@fluex/fluexgl-dsp";

const microphone = await audioDevice.createInputChannel(null, "Microphone");

microphone.addEffect(new Compressor({ threshold: -24, ratio: 4 }));
microphone.send(audioDevice.getMasterChannel());

// Later on, switch to another microphone without rebuilding anything.
const inputs = await listAudioInputDevices();
await microphone.setInputDevice(inputs[1]);
```

- - -

## Signal flow

```
input device / MediaStream -> MediaStreamAudioSourceNode -> input -> [effects] -> panner -> analyser -> gain -> output
```

Everything from ``input`` onwards is the regular [``Channel``](./Channel.md) chain. Switching the device only replaces the ``MediaStreamAudioSourceNode``.

The channel is **not connected to anything by default**. Call ``send()`` to hear it. Monitoring a microphone through speakers can cause feedback, so use headphones while testing.

- - -

## Constructor
Constructs a new input channel. No device is opened yet: call ``setInputDevice()`` or ``setMediaStream()`` afterwards. [``audioDevice.createInputChannel()``](./AudioDevice.md#methods) does both in one call.

```ts
new InputChannel(context: AudioContext, label?: string, options?: Partial<InputChannelOptions>): InputChannel;
```

### Arguments
- ``context``: ``AudioContext`` - The AudioContext used to create the audio nodes. Use the context of your [``AudioDevice``](./AudioDevice.md).
- ``label?``: ``string`` - Optional label. Defaults to ``"Input"``.
- ``options?``: [``Partial<InputChannelOptions>``](../interfaces/InputChannelOptions.md) - Capture options. Echo cancellation, noise suppression and automatic gain control are **disabled** by default, so effects receive the unprocessed signal.

- - -

## Properties

All properties of [``Channel``](./Channel.md), plus:

### ``stream: MediaStream | null``
The stream this channel currently receives audio from. ``null`` when closed.

### ``sourceNode: MediaStreamAudioSourceNode | null``
The node that feeds ``stream`` into ``input``. ``null`` when closed.

### ``deviceInfo: MediaDeviceInfo | null``
The input device that is currently open. ``null`` when closed, when the browser does not expose the device, or when an external stream is used (``setMediaStream()``).

### ``options: InputChannelOptions``
The resolved capture options. Changes are applied the next time a device is opened. See [``InputChannelOptions``](../interfaces/InputChannelOptions.md).

- - -

## Methods

All methods of [``Channel``](./Channel.md), plus:

### ``setInputDevice(device?: MediaDeviceInfo | string | null): Promise<boolean>``
Opens (or switches to) the given input device. Passing nothing, ``null`` or ``"default"`` uses the system's default input device. The new device is opened before the old one is released, so the gap is as short as possible. Fires ``input-device-changed`` on success.

When this method is called again before the previous call finished, only the last call wins; earlier streams are stopped right away.

#### Arguments
- ``device?``: ``MediaDeviceInfo | string | null`` - The input device (from [``listAudioInputDevices()``](../helpers/listAudioInputDevices.md)), or its ``deviceId``.

#### Returns (promised)
- ``boolean`` - ``true`` when the device has been opened, otherwise ``false``.

#### Errors
- ``ERROR:FLUEXGL-DSP@0014`` (``ErrorCodes.CHANNEL_NOT_INITIALIZED``) - The channel has no audio nodes.
- ``ERROR:FLUEXGL-DSP@0023`` (``ErrorCodes.INPUT_DEVICE_UNAVAILABLE``) - Permission was denied, the device does not exist (anymore), or the stream could not be connected to the ``AudioContext`` (Firefox does not support input devices with a different sample rate than the context).

### ``open(device?: MediaDeviceInfo | string | null): Promise<boolean>``
Alias for ``setInputDevice(device)``.

### ``setMediaStream(stream: MediaStream): boolean``
Uses an existing ``MediaStream`` as input instead of opening an input device. Use this for audio that does not come from a local device, such as the voice of another player received over WebRTC. Combined with [``SpatialAudioSource.attachChannel()``](./SpatialAudioSource.md#methods) this gives positioned voice chat (proximity chat). Fires ``input-device-changed`` with ``current: null``.

- The stream stays owned by the caller: ``close()`` does **not** stop its tracks.
- When the stream ends, ``input-device-lost`` is fired, but the channel never falls back to the local microphone.
- Chromium based browsers do not deliver remote WebRTC audio to the Web Audio graph unless the stream is also played by a media element. The channel takes care of this with a muted, invisible ``<audio>`` element.
- Cancels pending ``setInputDevice()`` calls.

#### Arguments
- ``stream``: ``MediaStream`` - A stream with at least one audio track.

#### Returns
- ``boolean`` - ``true`` when the stream is connected, otherwise ``false``.

#### Errors
- ``ERROR:FLUEXGL-DSP@0014`` (``ErrorCodes.CHANNEL_NOT_INITIALIZED``) - The channel has no audio nodes.
- ``ERROR:FLUEXGL-DSP@0023`` (``ErrorCodes.INPUT_DEVICE_UNAVAILABLE``) - The stream has no audio tracks, or could not be connected to the ``AudioContext``.

### ``close(): void``
Stops capturing. Tracks of a device opened by this channel are stopped (which turns off the browser's recording indicator); tracks of an external stream are left alone. The channel itself, its effects and its sends stay intact, so it can be opened again later.

#### Arguments
No arguments

#### Returns
- ``void``

### ``setMuted(muted: boolean): InputChannel``
Mutes or unmutes the input without closing it, by disabling the audio tracks.

#### Arguments
- ``muted``: ``boolean``

#### Returns
- ``InputChannel`` - The same channel.

### ``addEventListener(event, callback): () => void``
Registers a callback for one of the events below.

#### Arguments
- ``event``: ``keyof InputChannelEventMap`` - ``"input-device-changed"`` or ``"input-device-lost"``.
- ``callback``: ``InputChannelEventMap[event]`` - See [``InputChannelEventMap``](../interfaces/InputChannelEventMap.md).

#### Returns
- ``() => void`` - A function that removes the listener again.

### ``removeEventListener(event, callback): InputChannel``
Removes a previously registered callback.

#### Returns
- ``InputChannel`` - The same channel.

- - -

## Events

Registered with ``addEventListener()``. See [``InputChannelEventMap``](../interfaces/InputChannelEventMap.md).

| Event | Payload | Fired when |
|---|---|---|
| ``"input-device-changed"`` | [``AudioDeviceChangedEvent``](../interfaces/AudioDeviceChangedEvent.md) | A device was opened with ``setInputDevice()``, or a stream was connected with ``setMediaStream()``. |
| ``"input-device-lost"`` | [``AudioDeviceLostEvent``](../interfaces/AudioDeviceLostEvent.md) | The device was unplugged, permission was revoked, or an external stream ended. Logs ``WARNING:FLUEXGL-DSP@0004`` (``WarningCodes.INPUT_DEVICE_LOST``). For a local device with ``fallbackToDefault`` enabled, the default input device is opened afterwards. |

- - -

## Getters and setters

### ``get isOpen(): boolean``
Whether the channel is currently receiving audio from a device or stream.

- - -

## Examples

### Example 1: a microphone with a device menu
```ts
import { listAudioInputDevices, watchAudioDevices } from "@fluex/fluexgl-dsp";

const microphone = await audioDevice.createInputChannel(null, "Microphone");
microphone.send(audioDevice.getMasterChannel());

const select = document.querySelector("select")!;

function render(inputs: MediaDeviceInfo[]) {
    select.replaceChildren(...inputs.map(input => new Option(input.label, input.deviceId)));
    select.value = microphone.deviceInfo?.deviceId ?? "";
}

render(await listAudioInputDevices());

select.addEventListener("change", () => microphone.setInputDevice(select.value));
watchAudioDevices(({ inputs }) => render(inputs));
```

### Example 2: speech-friendly capture
Echo cancellation, noise suppression and automatic gain control are disabled by default, which is what you want for music and effects. For speech, turn them on:

```ts
const voice = await audioDevice.createInputChannel(null, "Voice", {
    echoCancellation: true,
    noiseSuppression: true,
    autoGainControl: true
});
```

### Example 3: the voice of another player (WebRTC)
```ts
import { InputChannel } from "@fluex/fluexgl-dsp";

peerConnection.addEventListener("track", (event) => {
    const voice = new InputChannel(audioDevice.getContext(), "Player 2");

    voice.setMediaStream(event.streams[0]);
    voice.send(audioDevice.getMasterChannel());

    voice.addEventListener("input-device-lost", () => {
        voice.unsendFromAll();
    });
});
```

See [Example 11: Proximity voice chat](../examples/11-proximity-voice-chat.md) for positioning voices in a game world.
