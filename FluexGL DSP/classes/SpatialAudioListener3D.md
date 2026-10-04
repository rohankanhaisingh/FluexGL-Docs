# Class ``SpatialAudioListener3D``

The point from which a [``SpatialAudioRenderer3D``](./SpatialAudioRenderer3D.md) renders the scene. Every renderer has exactly one listener.

Uses a right-handed coordinate system with y up and -z forward by default, the same as Web Audio and three.js. A camera can be followed by copying its world position and direction every frame.

For 2D scenes, use [``SpatialAudioListener``](./SpatialAudioListener.md).

## Example

```ts
const renderer = new SpatialAudioRenderer3D(audioDevice);

// First-person controls with yaw and pitch:
renderer.listener
    .setPosition(player.x, player.y + 1.7, player.z)
    .setYawPitch(player.yaw, player.pitch);

// Or follow a three.js camera:
camera.getWorldDirection(forward);
renderer.listener
    .setPosition(camera.position.x, camera.position.y, camera.position.z)
    .setOrientation(forward, camera.up);
```

- - -

## Constructor

```ts
new SpatialAudioListener3D(options?: Partial<SpatialAudioListener3DOptions>): SpatialAudioListener3D;
```

You normally do not construct a listener yourself: the renderer creates one. Use ``renderer.setSpatialAudioListener()`` to replace it.

### Arguments
- ``options?``: [``Partial<SpatialAudioListener3DOptions>``](../interfaces/SpatialAudioListener3DOptions.md) - Initial position, forward and up direction.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the listener. Should NOT be changed.

### ``position: Vector3``
World position of the listener. Defaults to ``{ x: 0, y: 0, z: 0 }``.

### ``forward: Vector3``
Normalized facing direction. Defaults to ``{ x: 0, y: 0, z: -1 }``. Set it with ``setOrientation()``, ``setYawPitch()`` or ``lookAt()`` rather than directly, so the internal basis stays up to date.

### ``up: Vector3``
Normalized up direction, kept perpendicular to ``forward``. Defaults to ``{ x: 0, y: 1, z: 0 }``.

- - -

## Methods

### ``setPosition(x: number, y: number, z: number): SpatialAudioListener3D``
Sets the world position.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``
- ``z``: ``number``

#### Returns
- ``SpatialAudioListener3D`` - The same listener. Can be used to stack methods.

### ``translate(dx: number, dy: number, dz: number): SpatialAudioListener3D``
Moves the listener by the given amount.

#### Arguments
- ``dx``: ``number``
- ``dy``: ``number``
- ``dz``: ``number``

#### Returns
- ``SpatialAudioListener3D`` - The same listener.

### ``setOrientation(forward: Vector3, up?: Vector3): SpatialAudioListener3D``
Sets the facing direction and the up direction. Neither has to be normalized. The up direction is made perpendicular to ``forward``. If both are parallel, a perpendicular direction is picked automatically.

#### Arguments
- ``forward``: [``Vector3``](../interfaces/Vector3.md) - Facing direction.
- ``up?``: [``Vector3``](../interfaces/Vector3.md) - Up direction. Defaults to the current ``up``.

#### Returns
- ``SpatialAudioListener3D`` - The same listener.

### ``setYawPitch(yaw: number, pitch?: number): SpatialAudioListener3D``
Sets the orientation from yaw (around the y-axis) and pitch (up/down), in radians. Yaw ``0`` and pitch ``0`` face -z. Positive yaw turns right, positive pitch looks up.

#### Arguments
- ``yaw``: ``number``
- ``pitch?``: ``number`` - Defaults to ``0``.

#### Returns
- ``SpatialAudioListener3D`` - The same listener.

### ``lookAt(x: number, y: number, z: number): SpatialAudioListener3D``
Rotates the listener so it faces the given point, with world up ``(0, 1, 0)``. Does nothing when the point equals the listener position.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``
- ``z``: ``number``

#### Returns
- ``SpatialAudioListener3D`` - The same listener.

### ``toLocal(position: Vector3): Vector3``
Converts a world position into listener space: ``x`` = right, ``y`` = up, ``z`` = forward.

#### Arguments
- ``position``: [``Vector3``](../interfaces/Vector3.md)

#### Returns
- [``Vector3``](../interfaces/Vector3.md)

- - -

## Getters and setters

### ``get right(): Vector3``
Unit vector pointing to the right of the listener, in world space (``forward x up``).

- - -

## Events

This class does not emit any events.
