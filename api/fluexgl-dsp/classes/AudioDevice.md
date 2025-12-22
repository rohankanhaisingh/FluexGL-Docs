# Class ``AudioDevice``

Represents an audio output device wrapper that owns an ``AudioContext`` and helps create and manage [``Channel``](./Channel.md) and [``Master``](./Master.md) routing objects. Usually constructed when calling [``ResolveDefaultAudioOutputDevice()``](../helpers/ResolveDefaultAudioOutputDevice.md).

## Example

```ts
import { AudioDevice } from "@fluex/fluexgl-dsp";

// MediaDeviceInfo is typically retrieved from navigator.mediaDevices.enumerateDevices()
const deviceInfo = myMediaDeviceInfo;

const device = new AudioDevice(deviceInfo);
const channel = device.CreateChannel();

// Create an additional master bus and set it as active
const master = device.CreateMasterChannel();
device.SetMasterChannel(master);
```

- - -

## Constructor
Constructs a new AudioDevice for the given output device and creates an internal ``AudioContext`` along with its default master channel.

```ts
new AudioDevice(public deviceInfo: MediaDeviceInfo): AudioDevice;
```

### Arguments
- ``deviceInfo``: ``MediaDeviceInfo`` - The selected output device information (from ``enumerateDevices()``).

- - -

## Properties

### ``id: string``
A unique id for this AudioDevice instance. Automatically generated when constructing the device. Should NOT be changed.

### ``timestamp: number``
Creation timestamp (``Date.now()``) for this AudioDevice instance.

### ``context: AudioContext``
The ``AudioContext`` owned by this device. Used for creating channels and master busses.

### ``masterChannel: Master``
The currently selected active [``Master``](./Master.md) channel for this device.

### ``masterChannels: Master[]``
All created [``Master``](./Master.md) channels owned by this device.

- - -

## Methods

### ``GetMasterChannel(): Master``
Returns the currently active [``Master``](./Master.md) channel.

#### Arguments
No arguments

#### Returns
- [``Master``](./Master.md)

### ``SetMasterChannel(channel: Master): void``
Sets the active [``Master``](./Master.md) channel. This is typically one of the entries in ``masterChannels``.

#### Arguments
- ``channel``: [``Master``](./Master.md) - The master channel to set as active.

#### Returns
- ``void``

### ``CreateMasterChannel(): Master``
Creates a new [``Master``](./Master.md) channel using this device’s ``AudioContext``, stores it in ``masterChannels``, and returns it.

#### Arguments
No arguments

#### Returns
- [``Master``](./Master.md)

### ``GetContext(): AudioContext``
Returns this device’s ``AudioContext``.

#### Arguments
No arguments

#### Returns
- ``AudioContext``

### ``CreateChannel(): Channel``
Creates and returns a new [``Channel``](./Channel.md) using this device’s ``AudioContext``.

#### Arguments
No arguments

#### Returns
- [``Channel``](./Channel.md)

## Events

This class does not emit custom events.

## Getters and setters

This class does not define public getters or setters.

## Examples

### Example 1: creating channels and routing them into the active master
```ts
const device = new AudioDevice(deviceInfo);

const a = device.CreateChannel();
const b = device.CreateChannel();

// Route a into b, and b into the active master
a.Send(b);
b.Send(device.GetMasterChannel());
```

### Example 2: creating multiple master busses
```ts
const device = new AudioDevice(deviceInfo);

const masterA = device.GetMasterChannel();
const masterB = device.CreateMasterChannel();

// Switch active master
device.SetMasterChannel(masterB);

// You can still keep references to older master channels
console.log(masterA.id, masterB.id);
```
