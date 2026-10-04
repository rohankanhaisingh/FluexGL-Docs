# Class ``SpatialAudioVoice``

The DSP chain that actually spatializes audio. Managed by [``SpatialAudioRenderer``](./SpatialAudioRenderer.md) and not meant to be created directly. It is not exported from the package root, but you can read voices through ``renderer.voices`` and ``source.voice``, for example for metering or visualization.

```
[source taps] -> input -> lowpass -> panner -> output (dry) -> master
                                            \-> reverbSend  -> reverb bus
```

A voice either belongs to one source (individual), or is shared by multiple far away sources in roughly the same direction (cluster). Every source keeps its own gain stage, so only the filter, panning and reverb send are shared.

The panner is a ``StereoPannerNode`` for the ``"stereo"`` panning model, or a ``PannerNode`` (``"equalpower"`` / ``"HRTF"``) for the 3D panning models. The ``PannerNode`` only handles direction; it uses ``rolloffFactor: 0``, so distance attenuation is left to the renderer.

## Example

```ts
for (const source of renderer.sources) {
    const voice = source.voice;

    if (!voice) continue; // inaudible

    console.log(source.label, voice.isCluster ? `cluster of ${voice.size}` : "own voice", voice.filter.frequency.value);
}
```

- - -

## Constructor

```ts
new SpatialAudioVoice(context: AudioContext, isCluster: boolean, panningModel: SpatialPanningModel, dryDestination: AudioNode, reverbDestination: AudioNode): SpatialAudioVoice;
```

Created by the renderer.

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing the voice.

### ``context: AudioContext``
The AudioContext of the voice.

### ``isCluster: boolean``
Whether this voice is shared by a cluster of sources.

### ``panningModel: SpatialPanningModel``
The [``SpatialPanningModel``](../interfaces/SpatialPanningModel.md) this voice was created with.

### ``members: Set<SpatialAudioSource>``
The sources rendered by this voice.

### ``input: GainNode``
Receives the audio of all members.

### ``filter: BiquadFilterNode``
Lowpass filter (air absorption). ``filter.frequency.value`` is the current cutoff.

### ``panner: StereoPannerNode | PannerNode``
The panner, depending on the panning model.

### ``output: GainNode``
Dry output, connected to the renderer's master channel.

### ``reverbSend: GainNode``
Reverb send, connected to the renderer's reverb bus. ``reverbSend.gain.value`` is the current send level.

### ``centroidDirection: Vector3``
Cluster centre as seen from the listener (unit vector in listener space). Only meaningful for cluster voices.

### ``centroidDistance: number``
Average distance of the cluster members to the listener. Only meaningful for cluster voices.

- - -

## Methods

These methods are used by the renderer.

### ``attach(source: SpatialAudioSource, crossfadeTime: number): void``
Connects the output of a source to this voice, fading it in.

### ``detach(source: SpatialAudioSource, crossfadeTime: number): void``
Fades a source out of this voice and disconnects it afterwards.

### ``has(source: SpatialAudioSource): boolean``
Whether the source is a member of this voice.

### ``setParameters(parameters: SpatialVoiceParameters, smoothing: number): void``
Applies cutoff, pan, direction and reverb send. A new voice jumps to its first values; later changes are smoothed with ``smoothing`` as time constant.

### ``dispose(crossfadeTime: number): void``
Detaches all members and disconnects the voice once the crossfade has finished.

- - -

## Getters and setters

### ``get size(): number``
Number of members.

- - -

## Events

This class does not emit any events.
