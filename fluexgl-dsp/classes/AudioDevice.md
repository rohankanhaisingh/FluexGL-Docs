# Class ``AudioDevice``

Represents an audio output device wrapper that owns an ``AudioContext`` and helps create and manage [``Channel``](./Channel.md) and [``Master``](./Master.md) routing objects. Usually constructed when calling [``resolveDefaultAudioOutputDevice()``](../helpers/ResolveDefaultAudioOutputDevice.md).

## Example

```ts
import { AudioDevice } from "@fluex/fluexgl-dsp";

// MediaDeviceInfo is typically retrieved from navigator.mediaDevices.enumerateDevices()
const deviceInfo = myMediaDeviceInfo;

const device = new AudioDevice(deviceInfo);
const channel = device.createChannel();

// Create an additional master bus and set it as active
const master = device.createMasterChannel();
device.setMasterChannel(master);
```

- - -

## Constructor
Constructs a new AudioDevice for the given output device and creates an internal ``AudioContext`` along with its default master channel.

```ts
new AudioDevice(deviceInfo: MediaDeviceInfo): AudioDevice;
```

### Arguments
- ``deviceInfo``: ``MediaDeviceInfo`` - The selected output device information (from ``enumerateDevices()``).

- - -

## Properties

### ``deviceInfo: MediaDeviceInfo``
The device information this instance was constructed with. Set via the constructor.

### ``id: string``
A unique id for this AudioDevice instance. Automatically generated when constructing the device. Should NOT be changed.

### ``timestamp: number``
Creation timestamp (``Date.now()``) for this AudioDevice instance.

### ``context: AudioContext``
The ``AudioContext`` owned by this device. Used for creating channels and master busses.

### ``masterChannel: Master``
The currently selected active [``Master``](./Master.md) channel for this device.

### ``masterChannels: Master[]``
All created [``Master``](./Master.md) channels owned by this device via ``createMasterChannel()``. Note that the default ``masterChannel`` created in the constructor is **not** automatically pushed into this array.

### ``sampleRate: number`` *(readonly)*
The sample rate of ``context``, captured at construction time.

### ``baseLatency: number`` *(readonly)*
The base latency of ``context``, captured at construction time.

### ``outputLatency: number`` *(readonly)*
The output latency of ``context``, captured at construction time.

### ``state: AudioContextState`` *(readonly)*
The state of ``context`` (e.g. ``"running"``, ``"suspended"``), captured at construction time.

### ``currentTime: number`` *(readonly)*
The ``currentTime`` of ``context``, captured at construction time. Because this is read once during construction, it does **not** update as time passes — use ``getContext().currentTime`` for a live value.

### ``maximumFrequency: number`` *(readonly)*
Half of ``context.sampleRate`` (the Nyquist frequency), captured at construction time.

- - -

## Methods

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

## Events

This class does not emit custom events.

## Getters and setters

This class does not define public getters or setters. Note that ``sampleRate``, ``baseLatency``, ``outputLatency``, ``state``, ``currentTime`` and ``maximumFrequency`` are plain ``readonly`` properties rather than getters, and are only computed once at construction time.

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
