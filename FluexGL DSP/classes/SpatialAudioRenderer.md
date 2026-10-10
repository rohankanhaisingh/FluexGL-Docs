# Class ``SpatialAudioRenderer``

Abstract base class of [``SpatialAudioRenderer2D``](./SpatialAudioRenderer2D.md) and [``SpatialAudioRenderer3D``](./SpatialAudioRenderer3D.md). Contains everything both renderers share: the output (its own master channel, or a bus you pass in), the reverb bus, the safety limiter, source management, fire-and-forget sounds (``playAt()``), distance attenuation, air absorption (lowpass), reverb sends, voice management, clustering and loop virtualization.

``SpatialAudioRenderer`` cannot be instantiated directly. Use ``SpatialAudioRenderer2D`` or ``SpatialAudioRenderer3D`` instead. Subclasses only have to convert a source position into listener space.

See [Spatial audio](../../Getting%20started/Spatial%20audio.md) for a conceptual overview.

## Signal flow

```
SpatialAudioSource (own gain: volume x distance attenuation)
        |  (crossfades when moving between voices)
        v
SpatialAudioVoice: lowpass -> panner -+-> source.bus, or the renderer's output (-> limiter)
  (own or shared)                     +-> reverb send -> reverbChannel -> output
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

- When ``options.output`` is given (a bus [``Channel``](./Channel.md) or a [``Master``](./Master.md)), the renderer sends everything there and does not create a master channel of its own.
- Otherwise, when ``target`` is an [``AudioDevice``](./AudioDevice.md), the renderer's master channel is created with ``audioDevice.createMasterChannel()``.
- Otherwise, when ``target`` is an ``AudioContext``, a new [``Master``](./Master.md) is created on that context.

The constructor also creates the reverb bus (``reverbChannel``), which is sent into the output. A [``Limiter``](../effects/Limiter.md) is attached to the renderer's own master channel unless ``limiter: false`` is passed. When an ``output`` is given, the limiter is only attached when ``limiter`` is ``true`` or an options object, since the output is then owned by your game. If ``clustering.mergeDistance`` is not larger than ``clustering.splitDistance``, a warning is logged and ``mergeDistance`` is set to ``splitDistance * 1.25``.

### Arguments
- ``target``: [``AudioDevice``](./AudioDevice.md) | ``AudioContext`` - The audio device or context of the renderer. Also where the renderer's master channel is created when no ``output`` is given.
- ``options?``: [``Partial<SpatialAudioRendererOptions>``](../interfaces/SpatialAudioRendererOptions.md) - Renderer configuration. Every field is optional.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the renderer. Should NOT be changed.

### ``label: string | null``
Custom label for this renderer. Taken from ``options.label``. Defaults to ``null``.

### ``context: AudioContext``
The AudioContext the renderer and all of its sources run on.

### ``master: Master | null``
The renderer's [``Master``](./Master.md) channel: its own master channel, or ``options.output`` when that is a ``Master``. ``null`` when ``options.output`` is a ``Channel``.

### ``output: Channel | Master``
Where the sound of the renderer goes: ``options.output``, or the renderer's own master channel. Voices of sources without a ``bus`` and the reverb bus end up here.

### ``reverbChannel: Channel``
A [``Channel``](./Channel.md) that receives the reverb sends of all voices, and is sent into ``output``. One reverb is shared by all buses. Its volume is set to ``0`` while no reverb effect is attached, so it does not add a dry copy of the signal.

### ``reverbEffect: Effector | null``
The effect currently attached to ``reverbChannel``. By default a [``Reverb``](../effects/Reverb.md) (``mix: 1``, ``roomSize: 0.6``, ``damping: 0.4``) is attached automatically on the first ``update()`` after WebAssembly has been initialized, unless ``setReverbEffect()`` has been called.

### ``limiter: Limiter | null``
The safety [``Limiter``](../effects/Limiter.md) on ``output``, which keeps many sources playing at once from clipping. ``null`` when disabled with ``{ limiter: false }``, and by default when an ``output`` is given.

### ``sources: SpatialAudioSource[]``
All [``SpatialAudioSource``](./SpatialAudioSource.md) instances added to this renderer.

### ``voices: SpatialAudioVoice[]``
All currently active [``SpatialAudioVoice``](./SpatialAudioVoice.md) instances (individual and clustered). Inaudible sources have no voice.

### ``options: ResolvedSpatialRendererOptions``
The resolved renderer options (all fields of [``SpatialAudioRendererOptions``](../interfaces/SpatialAudioRendererOptions.md) except ``clustering``, ``limiter`` and ``output``, with defaults filled in). Can be changed at runtime; changes apply on the next ``update()``.

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

It then assigns voices (see [Clustering](#clustering)), applies the combined parameters to every voice, and suspends or resumes looping sounds (see [Loop virtualization](#loop-virtualization)). Call this once per frame from your own loop, or use ``start()``.

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

### ``playAt(sound: Sound, position: Vector2 | Vector3, options?: Partial<SoundPlayAtOptions>): SoundInstance | null``
Plays a [``Sound``](./Sound.md) at a position, fire-and-forget. Meant for sounds that do not belong to a long-living object: explosions, impacts, collisions, bullet hits, and ambience of objects (with ``loop: true``).

The spatial source is borrowed from a pool (up to 256 sources are kept) and returned once the sound has ended, so playing a sound does not create new source nodes every time. The new source is rendered immediately without fading in, so the attack of the sound stays intact.

A one-shot that is inaudible when it starts (too far away) is skipped entirely: no audio node is created and ``null`` is returned. Pass ``cull: false`` to start it anyway. Looping sounds are never skipped; instead they are suspended while nobody can hear them (see [Loop virtualization](#loop-virtualization)).

#### Arguments
- ``sound``: [``Sound``](./Sound.md) - The sound to play.
- ``position``: [``Vector2``](../interfaces/Vector2.md) | [``Vector3``](../interfaces/Vector3.md) - World position. ``z`` is ignored in 2D.
- ``options?``: [``Partial<SoundPlayAtOptions>``](../interfaces/SoundPlayAtOptions.md) - Options of the instance (volume, pitch, loop, ...) and of the source (``bus``, ``refDistance``, ``reverbSendFactor``, ...).

#### Returns
- [``SoundInstance``](./SoundInstance.md) | ``null`` - ``null`` when the sound was culled, or skipped because of its ``minInterval`` or ``maxInstances``. Use ``instance.setPosition()`` to let it follow a moving object and ``instance.stop()`` to stop a loop.

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
Returns counters that show how the renderer is doing: sources, audible sources, virtual sources, voices, clusters, pooled voices and suspended loops. Useful for debugging and for tuning ``maxVoices``.

#### Arguments
No arguments

#### Returns
- [``SpatialRendererStats``](../interfaces/SpatialRendererStats.md)

### ``dispose(): void``
Stops the loop, removes and disposes all sources (which stops their sounds), disposes all voices and pooled sources, disposes the reverb bus and removes the limiter from ``output`` when the renderer attached one. A renderer can not be used after ``dispose()``; ``playAt()`` returns ``null``.

#### Arguments
No arguments

#### Returns
- ``void``

- - -

## Clustering

Every ``update()`` assigns voices in these steps (steps 1 to 4 only when ``clustering.enabled`` is ``true``):

1. Members that no longer fit their cluster are removed. A member must stay further than ``splitDistance`` and within ``maxAngle x 1.25`` and ``maxDistanceRatio x 1.25`` of the cluster centre (the extra 25% prevents flapping), and still be on the same bus as the cluster.
2. Loose sources further than ``mergeDistance`` join the best fitting existing cluster on the same bus (smallest angle), if it has fewer than ``maxMembers`` members. Joining a cluster costs no voice, so virtual sources can join too.
3. Remaining loose sources on the same bus that are within ``maxAngle`` and ``maxDistanceRatio`` of each other form new clusters, as long as the voice budget allows (see below).
4. Clusters with fewer than two members are dissolved.
5. Inaudible sources lose their voice, and sources whose ``bus`` changed get a new one.
6. If more voices are in use than ``maxVoices`` (for example after a cluster split up), the quietest voices are released.
7. The remaining audible sources get their own voice, loudest first, as long as the voice budget allows.
8. Empty voices go to a pool for reuse.

Cluster forming does not compare every pair of sources. Candidates are sorted by azimuth and only compared with neighbours within an azimuth window that provably contains every match; sources more than 60 degrees above or below the listener are compared with all others. This keeps ``update()`` fast with many sources (about 0.4 ms per frame for 1000 sources in 2D).

Angles are measured in 3D between directions as seen from the listener, so clustering works the same in 2D and 3D. A cluster voice uses the loudness-weighted average of its members' parameters (the cutoff is averaged logarithmically). Moving a source between voices always uses a short crossfade (``crossfadeTime``).

Because the gain stays per source, clustering only approximates the filter, panning and reverb of far away sources, which are the sources where that is hard to hear.

- - -

## Buses

By default everything a renderer plays ends up in one place: its own master channel. In a game you usually want separate volumes and effects for, for example, effects, entities and ambience. There are two levels:

- ``options.output`` sets where the renderer sends its sound and its reverb, for example an "Entities" bus [``Channel``](./Channel.md).
- ``source.bus`` (or the ``bus`` option of ``createSource()`` and ``playAt()``) sends one source to another bus, for example "Ambience".

```ts
const master = audioDevice.getMasterChannel();
const effects = audioDevice.createChannel("Effects");
const entities = audioDevice.createChannel("Entities");
const ambience = audioDevice.createChannel("Ambience");

for (const bus of [effects, entities, ambience]) bus.send(master);

const world = new SpatialAudioRenderer3D(audioDevice, { output: entities });

const enemySource = world.createSource({ position: enemy.position });        // -> Entities
world.playAt(sounds.explosion, position, { bus: effects });                 // -> Effects
world.playAt(sounds.torchLoop, torch.position, { bus: ambience, loop: true }); // -> Ambience
```

Sources only share a voice (a cluster) with sources on the same bus, so changing the volume of a bus never affects sources on another bus. Voices are pooled across buses: a pooled voice is reconnected to the bus it is reused for. The reverb is shared and returns into ``output``.

- - -

## Loop virtualization

Looping [``Sound``](./Sound.md)s keep an ``AudioBufferSourceNode`` alive for as long as they play. In a game world with many ambience loops, most of them are too far away to hear.

On every ``update()``, a source that has been without a voice (inaudible, or virtual because of the voice budget) for ``options.loopVirtualizationDelay`` seconds (default ``0.5``) has its looping instances suspended: their ``AudioBufferSourceNode``s are stopped and released. Once the source gets a voice again, the loops are resumed at the position they would have been at, so they stay in time. The delay keeps loops at the edge of hearing from being restarted every frame.

This applies to loops started with ``playAt()`` and with [``source.play()``](./SpatialAudioSource.md). One-shots are never suspended. Loops of [``AudioClip``](./AudioClip.md)s attached to a source are not virtualized. Set ``loopVirtualizationDelay: Infinity`` to disable it, and use ``getStats().suspendedLoops`` to see it work.

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
