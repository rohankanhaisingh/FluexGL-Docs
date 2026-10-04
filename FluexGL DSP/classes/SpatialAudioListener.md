# Class ``SpatialAudioListener``

The point from which a [``SpatialAudioRenderer2D``](./SpatialAudioRenderer2D.md) renders the scene. Every renderer has exactly one listener. All sources are attenuated, filtered and panned relative to this listener.

The rotation is optional. With the default rotation of ``0`` the listener faces up on the screen, so sources on the left and right of the screen are panned left and right.

For 3D scenes, use [``SpatialAudioListener3D``](./SpatialAudioListener3D.md).

## Example

```ts
const renderer = new SpatialAudioRenderer2D(audioDevice);

// Every renderer already has a listener:
renderer.listener.setPosition(player.x, player.y);

// Optional: rotate the listener, for example towards the mouse.
renderer.listener.lookAt(mouse.x, mouse.y);
```

- - -

## Constructor

```ts
new SpatialAudioListener(options?: Partial<SpatialAudioListenerOptions>): SpatialAudioListener;
```

You normally do not construct a listener yourself: the renderer creates one. Use ``renderer.setSpatialAudioListener()`` to replace it.

### Arguments
- ``options?``: [``Partial<SpatialAudioListenerOptions>``](../interfaces/SpatialAudioListenerOptions.md) - Initial position, rotation and y-axis direction.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the listener. Should NOT be changed.

### ``position: Vector2``
World position of the listener. Defaults to ``{ x: 0, y: 0 }``.

### ``rotation: number``
Facing direction in radians, clockwise. ``0`` means facing up on the screen. Defaults to ``0``.

### ``yAxis: SpatialYAxisDirection``
Direction of the y-axis on screen. Kept in sync with the renderer this listener belongs to. Defaults to ``"down"``. See [``SpatialYAxisDirection``](../interfaces/SpatialYAxisDirection.md).

- - -

## Methods

### ``setPosition(x: number, y: number): SpatialAudioListener``
Sets the world position.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``

#### Returns
- ``SpatialAudioListener`` - The same listener. Can be used to stack methods.

### ``translate(dx: number, dy: number): SpatialAudioListener``
Moves the listener by the given amount.

#### Arguments
- ``dx``: ``number``
- ``dy``: ``number``

#### Returns
- ``SpatialAudioListener`` - The same listener.

### ``setRotation(rotation: number): SpatialAudioListener``
Sets the facing direction in radians, clockwise, where ``0`` faces up on the screen.

#### Arguments
- ``rotation``: ``number``

#### Returns
- ``SpatialAudioListener`` - The same listener.

### ``lookAt(x: number, y: number): SpatialAudioListener``
Rotates the listener so it faces the given point. Does nothing when the point equals the listener position.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``

#### Returns
- ``SpatialAudioListener`` - The same listener.

### ``toLocal(position: Vector2): Vector3``
Converts a world position into listener space: ``x`` = right, ``y`` = up (always ``0`` in 2D), ``z`` = forward.

#### Arguments
- ``position``: [``Vector2``](../interfaces/Vector2.md)

#### Returns
- [``Vector3``](../interfaces/Vector3.md)

- - -

## Getters and setters

### ``get forward(): Vector2``
Unit vector of the facing direction, in world space.

### ``get right(): Vector2``
Unit vector pointing to the right of the listener, in world space.

- - -

## Events

This class does not emit any events.
