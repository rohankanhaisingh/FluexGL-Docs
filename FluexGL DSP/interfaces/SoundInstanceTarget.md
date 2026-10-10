# SoundInstanceTarget

Where a [``SoundInstance``](../classes/SoundInstance.md) plays. Only needed when calling [``sound.createInstance()``](../classes/Sound.md) directly; [``sound.play()``](../classes/Sound.md), [``source.play()``](../classes/SpatialAudioSource.md) and [``renderer.playAt()``](../classes/SpatialAudioRenderer.md) build it for you.

```ts
interface SoundInstanceTarget {
    context: BaseAudioContext;
    destination: AudioNode;
    options?: Partial<SoundPlayOptions>;
    owner?: SpatialAudioSource | null;
    exclusive?: boolean;
    release?: (() => void) | null;
}
```

## Properties
- `context`: `BaseAudioContext` - The context to create the audio nodes on.
- `destination`: `AudioNode` - The node the instance plays into.
- `options?`: [`Partial<SoundPlayOptions>`](./SoundPlayOptions.md) - Options of this instance.
- `owner?`: [`SpatialAudioSource`](../classes/SpatialAudioSource.md) | `null` - The spatial source the instance is heard through. Looping instances of an owner are suspended by the renderer while the owner has no voice.
- `exclusive?`: `boolean` - Whether the instance has the owner to itself (``playAt()``). The owner then carries the volume, so the renderer can rank it against other sources, and the owner's ``input`` gain is used to fade out. Otherwise the instance gets a ``GainNode`` of its own.
- `release?`: `() => void` | `null` - Called once the instance has ended. ``playAt()`` uses it to return the source to its pool.
