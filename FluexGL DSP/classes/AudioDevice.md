# Class ``AudioDevice``

Represents an audio output device wrapper that owns an ``AudioContext`` and helps create and manage [``Channel``](./Channel.md), [``Master``](./Master.md) and [``InputChannel``](./InputChannel.md) routing objects. Usually constructed when calling [``resolveDefaultAudioOutputDevice()``](../helpers/resolveDefaultAudioOutputDevice.md).

The output device can be switched at runtime with ``setOutputDevice()``, without recreating the ``AudioContext``. Master channels, channels, effects and audio clips keep working, so there is no need to reload the page.

## Example

```ts
import { listAudioOutputDevices } from "@fluex/fluexgl-dsp";

const device = await pipeline.resolveDefaultAudioOutputDevice();
const channel = device.createChannel();

channel.send(device.getMasterChannel());

// Switch to another output device while audio keeps playing.
const outputs = await listAudioOutputDevices();
await device.setOutputDevice(outputs[1]);

// Open the default microphone on its own channel.
const microphone = await device.createInputChannel(null, "Microphone");
```

- - -

## Constructor
Constructs a new AudioDevice and creates an internal ``AudioContext`` along with its default master channel. When ``deviceInfo`` is an output device other than the system's default, the ``AudioContext`` is created on that device (``sinkId``). If that fails, or the browser does not support it, the default output device is used and a warning is logged.

The constructor also starts listening for ``devicechange`` events of ``navigator.mediaDevices``, which powers the ``devices-changed`` and ``output-device-lost`` events. Call ``close()`` to stop listening.

```ts
new AudioDevice(deviceInfo?: MediaDeviceInfo | null): AudioDevice;
```

### Arguments
- ``deviceInfo?``: ``MediaDeviceInfo | null`` - The output device to play on (from [``listAudioOutputDevices()``](../helpers/listAudioOutputDevices.md) or ``enumerateDevices()``). Defaults to ``null``, which uses the system's default output device.

- - -

## Properties

### ``deviceInfo: MediaDeviceInfo | null``
The output device this instance is currently playing on. Set via the constructor and updated by ``setOutputDevice()``. ``null`` when unknown.

### ``id: string``
A unique id for this AudioDevice instance. Automatically generated when constructing the device. Should NOT be changed.

### ``timestamp: number``
Creation timestamp (``Date.now()``) for this AudioDevice instance.

### ``context: AudioContext``
The ``AudioContext`` owned by this device. Used for creating channels and master busses. Stays the same when the output device is switched.

### ``masterChannel: Master``
The currently selected active [``Master``](./Master.md) channel for this device.

### ``masterChannels: Master[]``
All created [``Master``](./Master.md) channels owned by this device via ``createMasterChannel()``. Note that the default ``masterChannel`` created in the constructor is **not** automatically pushed into this array.

### ``inputChannels: InputChannel[]``
All [``InputChannel``](./InputChannel.md) instances created via ``createInputChannel()``. Closed automatically by ``close()``.

### ``sampleRate: number`` *(readonly)*
The sample rate of ``context``.

### ``maximumFrequency: number`` *(readonly)*
Half of ``context.sampleRate`` (the Nyquist frequency).

- - -

## Methods

### ``setOutputDevice(device?: MediaDeviceInfo | string | null): Promise<boolean>``
Switches the output device this audio device plays on, using ``AudioContext.setSinkId()``. The ``AudioContext`` and everything built on it stay intact. Passing nothing, ``null`` or ``"default"`` switches to the system's default output device. Fires ``output-device-changed`` on success.

Requires a browser that supports ``AudioContext.setSinkId()`` (Chromium based browsers). Check ``AudioDevice.supportsOutputDeviceSelection`` first.

#### Arguments
- ``device?``: ``MediaDeviceInfo | string | null`` - The output device, or its ``deviceId``. Must be an ``"audiooutput"`` device.

#### Returns (promised)
- ``boolean`` - ``true`` when the output device has been switched, otherwise ``false``.

#### Errors and warnings
- ``WARNING:FLUEXGL-DSP@0002`` (``WarningCodes.OUTPUT_DEVICE_SWITCH_UNSUPPORTED``) - The browser does not support ``AudioContext.setSinkId()``. Returns ``false``.
- ``ERROR:FLUEXGL-DSP@0024`` (``ErrorCodes.OUTPUT_DEVICE_UNAVAILABLE``) - The given device is not an output device, or the browser refused to switch to it (for example because it was unplugged).

### ``createInputChannel(device?: MediaDeviceInfo | string | null, label?: string, options?: Partial<InputChannelOptions>): Promise<InputChannel>``
Creates a new [``InputChannel``](./InputChannel.md) on this device's ``AudioContext``, opens the given input device on it, and stores it in ``inputChannels``. Passing nothing, ``null`` or ``"default"`` opens the system's default input device.

The input channel is not connected to anything by default. Use ``send()`` to route it to a channel or master channel.

#### Arguments
- ``device?``: ``MediaDeviceInfo | string | null`` - The input device, or its ``deviceId``.
- ``label?``: ``string`` - Optional label. Defaults to ``"Input"``.
- ``options?``: [``Partial<InputChannelOptions>``](../interfaces/InputChannelOptions.md) - Capture options, such as echo cancellation.

#### Returns (promised)
- [``InputChannel``](./InputChannel.md) - Also returned when the device could not be opened; check ``inputChannel.isOpen``.

### ``removeInputChannel(inputChannel: InputChannel): void``
Closes the input channel, removes all its sends (to channels and master channels) and removes it from ``inputChannels``.

#### Arguments
- ``inputChannel``: [``InputChannel``](./InputChannel.md)

#### Returns
- ``void``

#### Errors
- ``ERROR:FLUEXGL-DSP@0012`` (``ErrorCodes.CHANNEL_NOT_FOUND``) - The input channel is not part of this audio device.

### ``close(): Promise<void>``
Closes all input channels, stops listening for device changes and closes the ``AudioContext``. The audio device cannot be used anymore afterwards.

#### Arguments
No arguments

#### Returns (promised)
- ``void``

### ``addEventListener(event, callback): () => void``
Registers a callback for one of the events below.

#### Arguments
- ``event``: ``keyof AudioDeviceEventMap`` - ``"output-device-changed"``, ``"output-device-lost"`` or ``"devices-changed"``.
- ``callback``: ``AudioDeviceEventMap[event]`` - See [``AudioDeviceEventMap``](../interfaces/AudioDeviceEventMap.md).

#### Returns
- ``() => void`` - A function that removes the listener again.

### ``removeEventListener(event, callback): AudioDevice``
Removes a previously registered callback.

#### Returns
- ``AudioDevice`` - The same device.

### ``getMasterChannel(): Master``
Returns the currently active [``Master``](./Master.md) channel.

#### Arguments
No arguments

#### Returns
- [``Master``](./Master.md)

### ``setMasterChannel(channel: Master): void``
Sets the active [``Master``](./Master.md) channel. This is typically one of the entries in ``masterChannels``. Logs an error and does nothing if the given channel is already the active one.

#### Arguments
- ``channel``: [``Master``](./Master.md) - The master channel to set as active.

#### Returns
- ``void``

### ``createMasterChannel(): Master``
Creates a new [``Master``](./Master.md) channel using this device’s ``AudioContext``, stores it in ``masterChannels``, and returns it.

#### Arguments
No arguments

#### Returns
- [``Master``](./Master.md)

### ``getContext(): AudioContext``
Returns this device’s ``AudioContext``.

#### Arguments
No arguments

#### Returns
- ``AudioContext``

### ``createChannel(label?: string): Channel``
Creates and returns a new [``Channel``](./Channel.md) using this device’s ``AudioContext``.

#### Arguments
- ``label?``: ``string`` - Optional label for the new channel.

#### Returns
- [``Channel``](./Channel.md)

- - -

## Events

Registered with ``addEventListener()``. See [``AudioDeviceEventMap``](../interfaces/AudioDeviceEventMap.md).

| Event | Payload | Fired when |
|---|---|---|
| ``"output-device-changed"`` | [``AudioDeviceChangedEvent``](../interfaces/AudioDeviceChangedEvent.md) | ``setOutputDevice()`` switched the output device. |
| ``"output-device-lost"`` | [``AudioDeviceLostEvent``](../interfaces/AudioDeviceLostEvent.md) | The selected output device was disconnected. The device then falls back to the default output device (which fires ``output-device-changed`` as well) and logs ``WARNING:FLUEXGL-DSP@0003`` (``WarningCodes.OUTPUT_DEVICE_LOST``). |
| ``"devices-changed"`` | [``AudioDeviceListChangedEvent``](../interfaces/AudioDeviceListChangedEvent.md) | An audio input or output device was connected or disconnected. |

When the system's default output device is used, the browser follows changes of the default device by itself, so ``output-device-lost`` is only fired for a specifically selected device.

- - -

## Getters and setters

### ``get outputDeviceId(): string``
The ``deviceId`` of the output device this audio device is currently playing on. An empty string means the system's default output device (also in browsers without ``setSinkId()`` support).

### ``get baseLatency(): number``
The live ``baseLatency`` of ``context``.

### ``get outputLatency(): number``
The live ``outputLatency`` of ``context``. Can change when the output device is switched.

### ``get state(): AudioContextState``
The live state of ``context`` (e.g. ``"running"``, ``"suspended"``, ``"closed"``).

### ``get currentTime(): number``
The live ``currentTime`` of ``context``.

### ``static get supportsOutputDeviceSelection(): boolean``
Whether the browser supports switching the output device of an ``AudioContext`` (``AudioContext.prototype.setSinkId``).

- - -

## Examples

### Example 1: creating channels and routing them into the active master
```ts
const device = new AudioDevice(deviceInfo);

const a = device.createChannel();
const b = device.createChannel();

// Route a into b, and b into the active master
a.send(b);
b.send(device.getMasterChannel());
```

### Example 2: creating multiple master busses
```ts
const device = new AudioDevice(deviceInfo);

const masterA = device.getMasterChannel();
const masterB = device.createMasterChannel();

// Switch active master
device.setMasterChannel(masterB);

// You can still keep references to older master channels
console.log(masterA.id, masterB.id);
```

### Example 3: an output device menu
```ts
import { AudioDevice, listAudioOutputDevices } from "@fluex/fluexgl-dsp";

const select = document.querySelector("select")!;

async function render() {
    const outputs = await listAudioOutputDevices();

    select.replaceChildren(...outputs.map(output => new Option(output.label, output.deviceId)));
    select.value = device.outputDeviceId || "default";
}

select.disabled = !AudioDevice.supportsOutputDeviceSelection;
select.addEventListener("change", () => device.setOutputDevice(select.value));

device.addEventListener("devices-changed", render);
device.addEventListener("output-device-changed", render);

await render();
```

### Example 4: reacting to an unplugged output device
```ts
device.addEventListener("output-device-lost", ({ lost }) => {
    showToast(`${lost?.label ?? "Output device"} was disconnected, switched to the default device.`);
});
```
