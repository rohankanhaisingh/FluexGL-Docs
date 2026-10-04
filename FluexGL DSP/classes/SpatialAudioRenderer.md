# Class ``SpatialAudioRenderer``

Abstract base class of [``SpatialAudioRenderer2D``](./SpatialAudioRenderer2D.md) and [``SpatialAudioRenderer3D``](./SpatialAudioRenderer3D.md). Contains everything both renderers share: the master channel, the reverb bus, the safety limiter, source management, distance attenuation, air absorption (lowpass), reverb sends, voice management and clustering.

``SpatialAudioRenderer`` cannot be instantiated directly. Use ``SpatialAudioRenderer2D`` or ``SpatialAudioRenderer3D`` instead. Subclasses only have to convert a source position into listener space.

See [Spatial audio](../../Getting%20started/Spatial%20audio.md) for a conceptual overview.

## Signal flow

```
SpatialAudioSource (own gain: volume x distance attenuation)
        |  (crossfades when moving between voices)
        v
SpatialAudioVoice: lowpass -> panner -+-> master (-> limiter)
  (own or shared)                     +-> reverb send -> reverbChannel -> master
```

Every source keeps its own gain stage, so the loudness of each source stays exact. A voice holds the more expensive part of the chain (lowpass filter, panner and reverb send). Close sources get their own voice. Far away sources in roughly the same direction share one voice (a cluster).

## Example

```ts
import { SpatialAudioRenderer2D } from "@fluex/fluexgl-dsp";

// SpatialAudioRenderer itself is abstract; use one of its subclasses.
const renderer = new SpatialAudioRenderer2D(audioDevice, {
    maxDistance: 2000,
    clustering: { splitDistance: 250, mergeDistance: 340 }
});

const source = renderer.createSource({ position: { x: 300, y: 0 } });
source.attachAudioClip(clip);
clip.play();

renderer.start();
```

- - -

## Constructor

```ts
new SpatialAudioRenderer2D(target: AudioDevice | AudioContext, options?: Partial<SpatialAudioRenderer2DOptions>);
new SpatialAudioRenderer3D(target: AudioDevice | AudioContext, options?: Partial<SpatialAudioRenderer3DOptions>);
```

The base constructor is shared by both subclasses.

- When ``target`` is an [``AudioDevice``](./AudioDevice.md), the renderer's master channel is created with ``audioDevice.createMasterChannel()``.
- When ``target`` is an ``AudioContext``, a new [``Master``](./Master.md) is created on that context.

The constructor also creates the reverb bus (``reverbChannel``) and, unless disabled, attaches a [``Limiter``](../effects/Limiter.md) to the master channel. If ``clustering.mergeDistance`` is not larger than ``clustering.splitDistance``, a warning is logged and ``mergeDistance`` is set to ``splitDistance * 1.25``.

### Arguments
- ``target``: [``AudioDevice``](./AudioDevice.md) | ``AudioContext`` - Where the renderer's master channel is created.
- ``options?``: [``Partial<SpatialAudioRendererOptions>``](../interfaces/SpatialAudioRendererOptions.md) - Renderer configuration. Every field is optional.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the renderer. Should NOT be changed.

### ``label: string | null``
Custom label for this renderer. Taken from ``options.label``. Defaults to ``null``.

### ``context: AudioContext``
The AudioContext the renderer and all of its sources run on.

### ``master: Master``
The renderer's own [``Master``](./Master.md) channel. All voices and the reverb bus end up here. Every renderer has its own master channel.

### ``reverbChannel: Channel``
A [``Channel``](./Channel.md) that receives the reverb sends of all voices, and is sent into ``master``. Its volume is set to ``0`` while no reverb effect is attached, so it does not add a dry copy of the signal.

### ``reverbEffect: Effector | null``
The effect currently attached to ``reverbChannel``. By default a [``Reverb``](../effects/Reverb.md) (``mix: 1``, ``roomSize: 0.6``, ``damping: 0.4``) is attached automatically on the first ``update()`` after WebAssembly has been initialized, unless ``setReverbEffect()`` has been called.

### ``limiter: Limiter | null``
The safety [``Limiter``](../effects/Limiter.md) on ``master``, which keeps many sources playing at once from clipping. ``null`` when disabled with ``{ limiter: false }``.

### ``sources: SpatialAudioSource[]``
All [``SpatialAudioSource``](./SpatialAudioSource.md) instances added to this renderer.

### ``voices: SpatialAudioVoice[]``
All currently active [``SpatialAudioVoice``](./SpatialAudioVoice.md) instances (individual and clustered). Inaudible sources have no voice.

### ``options: ResolvedSpatialRendererOptions``
The resolved renderer options (all fields of [``SpatialAudioRendererOptions``](../interfaces/SpatialAudioRendererOptions.md) except ``clustering`` and ``limiter``, with defaults filled in). Can be changed at runtime; changes apply on the next ``update()``.

### ``clustering: SpatialClusteringOptions``
The resolved [``SpatialClusteringOptions``](../interfaces/SpatialClusteringOptions.md). Can be changed at runtime, for example ``renderer.clustering.enabled = false``.

- - -

## Methods

### ``update(): this``
Updates every source and voice. For each source the renderer:

1. Calculates its position relative to the listener ([``SpatialSourceState``](../interfaces/SpatialSourceState.md)).
2. Applies ``volume x attenuation`` to the source's own gain.
3. Marks it audible or inaudible (with hysteresis: a silent source becomes audible again at ``2 x silenceThreshold``).
4. Calculates its lowpass cutoff, pan/direction and reverb send.

It then assigns voices (see [Clustering](#clustering)) and applies the combined parameters to every voice. Call this once per frame from your own loop, or use ``start()``.

#### Arguments
No arguments

#### Returns
- ``this`` - The same renderer. Can be used to stack methods.

### ``start(): this``
Starts calling ``update()`` every animation frame (``requestAnimationFrame``), or every 16 ms when ``requestAnimationFrame`` is unavailable. Does nothing if already running.

#### Arguments
No arguments

#### Returns
- ``this`` - The same renderer.

### ``stop(): this``
Stops the loop started by ``start()``.

#### Arguments
No arguments

#### Returns
- ``this`` - The same renderer.

### ``createSource(options?: Partial<SpatialAudioSourceOptions>): SpatialAudioSource``
Creates a new [``SpatialAudioSource``](./SpatialAudioSource.md) and adds it to this renderer.

#### Arguments
- ``options?``: [``Partial<SpatialAudioSourceOptions>``](../interfaces/SpatialAudioSourceOptions.md) - Source configuration.

#### Returns
- [``SpatialAudioSource``](./SpatialAudioSource.md) - The created source.

### ``addSource(source: SpatialAudioSource): this``
Adds an existing source to this renderer and initializes its audio nodes on the renderer's ``AudioContext``. If the source belongs to another renderer, it is removed from that renderer first (without being disposed). Does nothing if the source is already part of this renderer.

#### Arguments
- ``source``: [``SpatialAudioSource``](./SpatialAudioSource.md) - The source to add.

#### Returns
- ``this`` - The same renderer.

### ``removeSource(source: SpatialAudioSource, dispose?: boolean): this``
Removes a source from this renderer and fades it out of its voice. When ``dispose`` is ``true`` (default), the source is disposed after the crossfade, which stops its clips and releases its audio nodes. When ``false``, the source is faded to silence and can be added to a renderer again.

#### Arguments
- ``source``: [``SpatialAudioSource``](./SpatialAudioSource.md) - The source to remove.
- ``dispose?``: ``boolean`` - Whether to dispose the source. Defaults to ``true``.

#### Returns
- ``this`` - The same renderer.

### ``computeSourceState(source: SpatialAudioSource): SpatialSourceState``
Calculates where a source is relative to the listener (local position, distance, azimuth, elevation, attenuation and normalized distance). Does not change anything; ``update()`` stores the result in ``source.state``.

#### Arguments
- ``source``: [``SpatialAudioSource``](./SpatialAudioSource.md) - The source to calculate the state for.

#### Returns
- [``SpatialSourceState``](../interfaces/SpatialSourceState.md)

### ``setReverbEffect(effect: Effector | null): this``
Replaces the effect on the reverb bus. Passing ``null`` removes the reverb and mutes the bus. After calling this, the default reverb is no longer attached automatically.

#### Arguments
- ``effect``: [``Effector``](./Effector.md) | ``null`` - The new reverb effect, or ``null`` to disable reverb.

#### Returns
- ``this`` - The same renderer.

### ``getClusters(): SpatialClusterInfo[]``
Returns how sources are currently grouped into voices. Useful for debugging and visualization.

#### Arguments
No arguments

#### Returns
- [``SpatialClusterInfo[]``](../interfaces/SpatialClusterInfo.md) - One entry per active voice.

### ``getStats(): SpatialRendererStats``
Returns counters that show how the renderer is doing: sources, audible sources, virtual sources, voices, clusters and pooled voices. Useful for debugging and for tuning ``maxVoices``.

#### Arguments
No arguments

#### Returns
- [``SpatialRendererStats``](../interfaces/SpatialRendererStats.md)

### ``dispose(): void``
Stops the loop, removes and disposes all sources, disposes all voices (including pooled ones) and disconnects the reverb bus from the master channel.

#### Arguments
No arguments

#### Returns
- ``void``

- - -

## Clustering

Every ``update()`` assigns voices in these steps (steps 1 to 4 only when ``clustering.enabled`` is ``true``):

1. Members that no longer fit their cluster are removed. A member must stay further than ``splitDistance`` and within ``maxAngle x 1.25`` and ``maxDistanceRatio x 1.25`` of the cluster centre (the extra 25% prevents flapping).
2. Loose sources further than ``mergeDistance`` join the best fitting existing cluster (smallest angle), if it has fewer than ``maxMembers`` members. Joining a cluster costs no voice, so virtual sources can join too.
3. Remaining loose sources that are within ``maxAngle`` and ``maxDistanceRatio`` of each other form new clusters, as long as the voice budget allows (see below).
4. Clusters with fewer than two members are dissolved.
5. Inaudible sources lose their voice.
6. If more voices are in use than ``maxVoices`` (for example after a cluster split up), the quietest voices are released.
7. The remaining audible sources get their own voice, loudest first, as long as the voice budget allows.
8. Empty voices go to a pool for reuse.

Cluster forming does not compare every pair of sources. Candidates are sorted by azimuth and only compared with neighbours within an azimuth window that provably contains every match; sources more than 60 degrees above or below the listener are compared with all others. This keeps ``update()`` fast with many sources (about 0.4 ms per frame for 1000 sources in 2D).

Angles are measured in 3D between directions as seen from the listener, so clustering works the same in 2D and 3D. A cluster voice uses the loudness-weighted average of its members' parameters (the cutoff is averaged logarithmically). Moving a source between voices always uses a short crossfade (``crossfadeTime``).

Because the gain stays per source, clustering only approximates the filter, panning and reverb of far away sources, which are the sources where that is hard to hear.

- - -

## Voice budget

``options.maxVoices`` caps the number of voices (a cluster counts as one). The default is ``64`` for ``SpatialAudioRenderer2D`` and ``32`` for ``SpatialAudioRenderer3D``, because HRTF panners are relatively expensive.

- When the budget is full, the quietest audible sources become **virtual** (``source.isVirtual``): they keep being tracked, but are not rendered.
- A virtual source (or a new cluster) takes over a voice only when it is at least 1.5x louder than the quietest voice in use. This margin keeps sources from switching back and forth.
- Loudness of a voice is the summed gain (volume x attenuation) of its sources.
- Released voices go to a pool and are reused, so voices (and HRTF panners) are not constantly created and destroyed.

## Performance

Every ``AudioParam`` automation call (``setTargetAtTime``, ``setValueAtTime``) is a message to the audio thread. With many sources, sending every parameter every frame floods the audio thread and causes glitches. The renderer therefore only sends a value when it changed by more than an inaudible amount: 0.5% for gains and cutoffs, 0.002 for pan, reverb send and direction components. In a benchmark with 1000 moving sources this reduced the automation calls from about 1500 to about 220 per frame.

- - -

## Getters and setters

### ``get isRunning(): boolean``
Whether the loop started by ``start()`` is running.

- - -

## Protected members

These are only relevant when extending ``SpatialAudioRenderer``.

### ``abstract toLocal(source: SpatialAudioSource): Vector3``
Converts the position of a source into listener space: ``x`` = right, ``y`` = up, ``z`` = forward.

### ``abstract panningModel: SpatialPanningModel``
The [``SpatialPanningModel``](../interfaces/SpatialPanningModel.md) used for newly created voices.

### ``rebuildVoices(): void``
Disposes all voices (with crossfades). New voices are created on the next ``update()``. Used by ``SpatialAudioRenderer3D.setPanningModel()``.

- - -

## Events

This class does not emit any events.
