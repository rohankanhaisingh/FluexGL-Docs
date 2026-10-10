# Class ``SoundInstance``

One playing instance of a [``Sound``](./Sound.md). Returned by [``sound.play()``](./Sound.md), [``source.play()``](./SpatialAudioSource.md) and [``renderer.playAt()``](./SpatialAudioRenderer.md).

Keep the instance when you want to control the sound while it plays: stop a loop, change its volume, or let it follow a moving object. For fire-and-forget sounds you can ignore it.

Once the instance has ended, its audio nodes are released and its spatial source (from ``playAt()``) is reused for other sounds. From then on the instance can no longer be controlled: ``source`` returns ``null`` and the setters do nothing.

## Example

```ts
// A rocket that whooshes while it flies.
const whoosh = renderer.playAt(rocketLoop, rocket.position, { loop: true });

function frame() {
    whoosh?.setPosition(rocket.x, rocket.y, rocket.z);
}

function onImpact() {
    whoosh?.stop(0.1);
    renderer.playAt(explosion, rocket.position);
}
```

- - -

## Constructor

Instances are created by [``Sound``](./Sound.md); you do not create them yourself.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when the instance is started.

### ``sound: Sound``
The [``Sound``](./Sound.md) this instance plays.

### ``startTime: number``
``AudioContext`` time at which the instance started (or will start, when ``when`` was given).

### ``loop: boolean``
Whether the instance loops.

- - -

## Methods

### ``stop(fadeTime?: number): void``
Stops the instance after a short fade. A fade time of ``0`` stops it immediately. A suspended loop (see [Loop virtualization](#loop-virtualization)) ends at once, since it has no audio node to fade. Stopping an instance that has already stopped does nothing.

#### Arguments
- ``fadeTime?``: ``number`` - Fade-out in seconds. Defaults to ``0.02``.

#### Returns
- ``void``

### ``setVolume(volume: number): SoundInstance``
Changes the volume of this instance (floored at ``0``). For an instance from ``playAt()`` this sets the volume of its spatial source, otherwise the instance's own gain is smoothed to the new value.

#### Arguments
- ``volume``: ``number``

#### Returns
- ``SoundInstance`` - The same instance.

### ``setPosition(x: number, y: number, z?: number): SoundInstance``
Moves the spatial source of an instance started with ``renderer.playAt()``. Does nothing for instances from ``sound.play()`` (not positioned) and ``source.play()`` (they move with their source), and for instances that have ended.

#### Arguments
- ``x``: ``number``
- ``y``: ``number``
- ``z?``: ``number`` - Defaults to the current ``z``.

#### Returns
- ``SoundInstance`` - The same instance.

### ``onEnded(callback: () => void): SoundInstance``
Calls ``callback`` once the instance has ended, either because the sound has finished or because it was stopped. If the instance has already ended, the callback is called immediately.

#### Arguments
- ``callback``: ``() => void``

#### Returns
- ``SoundInstance`` - The same instance.

### ``suspend(): void``
Releases the ``AudioBufferSourceNode`` of a looping instance. The loop keeps running "virtually". Does nothing for one-shots. Called by the renderer; see [Loop virtualization](#loop-virtualization).

#### Returns
- ``void``

### ``resume(): void``
Recreates the ``AudioBufferSourceNode`` of a suspended loop, at the position the loop would have been at. Called by the renderer.

#### Returns
- ``void``

- - -

## Getters and setters

### ``get isPlaying(): boolean``
``false`` once the instance has been stopped or has ended. A suspended loop counts as playing.

### ``get isSuspended(): boolean``
Whether the instance is a loop whose audio node is released because nobody can hear it.

### ``get volume(): number``
The current volume of this instance, including the random ``volumeVariation`` of its sound.

### ``get source(): SpatialAudioSource | null``
The [``SpatialAudioSource``](./SpatialAudioSource.md) this instance is heard through. ``null`` for instances from ``sound.play()``, and after the instance has ended.

### ``get loudness(): number``
The loudness used by ``steal: "quietest"``: the rendered gain (volume x distance attenuation) for spatial instances, the volume otherwise.

- - -

## Loop virtualization

A looping instance on a spatial source keeps an ``AudioBufferSourceNode`` alive for as long as it plays. With hundreds of ambience loops spread over a map, most of them are too far away to hear.

When a source has been without a voice for ``loopVirtualizationDelay`` seconds (default ``0.5``), the renderer suspends its looping instances: the ``AudioBufferSourceNode`` is stopped and released. A source has no voice when it is inaudible, or virtual because of the [voice budget](./SpatialAudioRenderer.md#voice-budget). Once the source gets a voice again, the renderer resumes them: a new node is started at the position the loop would have been at, so the loop stays in time. A loop of 2 seconds that was suspended 1.1 seconds after it started continues at 1.1 seconds.

One-shots are never suspended; they are short enough to play out. Set ``loopVirtualizationDelay: Infinity`` in the [renderer options](../interfaces/SpatialAudioRendererOptions.md) to disable it. ``renderer.getStats().suspendedLoops`` shows how many loops are suspended.

Loops of [``AudioClip``](./AudioClip.md)s attached to a source are not virtualized. Use ``Sound`` for ambience loops in a game world.

- - -

## Events

This class does not emit any events. Use ``onEnded()``.
