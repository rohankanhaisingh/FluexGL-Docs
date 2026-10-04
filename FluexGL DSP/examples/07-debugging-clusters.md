# Example 07: Debugging and tuning clusters

Shows what the spatial renderer is doing: where every source is as heard by the listener, which sources share a voice, and how to tune clustering.

## What it shows
- [``source.state``](../interfaces/SpatialSourceState.md): distance, angle and attenuation of a source.
- [``renderer.getClusters()``](../interfaces/SpatialClusterInfo.md): which sources share a voice.
- [``source.voice``](../classes/SpatialAudioVoice.md): the live filter, panning and reverb values.
- Tuning [``SpatialClusteringOptions``](../interfaces/SpatialClusteringOptions.md) at runtime.

## Logging the state of a source

```ts
renderer.update();

const state = source.state;

if (state) {
    console.log({
        distance: state.distance.toFixed(1),
        azimuth: (state.azimuth * 180 / Math.PI).toFixed(0) + " deg",   // 0 = in front, 90 = right
        elevation: (state.elevation * 180 / Math.PI).toFixed(0) + " deg", // 3D only
        attenuation: state.attenuation.toFixed(3),                        // 0..1
        audible: source.audible,
        voice: source.voice ? (source.voice.isCluster ? `cluster of ${source.voice.size}` : "own") : "none (silent)",
        cutoff: source.voice?.filter.frequency.value.toFixed(0) + " Hz",
        reverbSend: source.voice?.reverbSend.gain.value.toFixed(2)
    });
}
```

## Drawing clusters on a 2D canvas

```ts
function hsl(id: string): string {
    let hash = 0;
    for (const char of id) hash = (hash * 31 + char.charCodeAt(0)) | 0;
    return `hsl(${Math.abs(hash) % 360}, 80%, 60%)`;
}

function drawAudioDebug(ctx: CanvasRenderingContext2D) {

    const { position } = renderer.listener;

    // Split (solid) and merge (dashed) distance around the listener.
    ctx.strokeStyle = "rgba(255, 255, 255, 0.3)";
    ctx.beginPath();
    ctx.arc(position.x, position.y, renderer.clustering.splitDistance, 0, Math.PI * 2);
    ctx.stroke();

    ctx.setLineDash([6, 6]);
    ctx.beginPath();
    ctx.arc(position.x, position.y, renderer.clustering.mergeDistance, 0, Math.PI * 2);
    ctx.stroke();
    ctx.setLineDash([]);

    // Sources, colored by voice. Grey = silent, red = own voice, any other color = a cluster.
    for (const source of renderer.sources) {

        const voice = source.voice;

        ctx.fillStyle = !voice ? "#555" : voice.isCluster ? hsl(voice.id) : "#f44";
        ctx.beginPath();
        ctx.arc(source.position.x, source.position.y, 8, 0, Math.PI * 2);
        ctx.fill();
    }

    // Summary.
    const clusters = renderer.getClusters();

    ctx.fillStyle = "#fff";
    ctx.fillText(`voices: ${renderer.voices.length}, clusters: ${clusters.filter(c => c.isCluster).length}, sources: ${renderer.sources.length}`, 10, 20);
}
```

## Tuning

All clustering options can be changed at runtime:

```ts
// Turn clustering off to hear the difference.
renderer.clustering.enabled = false;

// Cluster more aggressively: wider angle, earlier merge.
renderer.clustering.maxAngle = Math.PI / 8;      // 22.5 degrees
renderer.clustering.splitDistance = 150;
renderer.clustering.mergeDistance = 200;

// Limit how many sources can share one voice.
renderer.clustering.maxMembers = 8;
```

| Symptom | Try |
|---|---|
| Sources switch between cluster and own voice too often | Increase the gap between ``splitDistance`` and ``mergeDistance`` |
| A cluster sounds like it comes from the wrong direction | Lower ``maxAngle`` |
| Near and far sources end up in one cluster | Lower ``maxDistanceRatio`` |
| Too many voices (CPU usage, especially with HRTF) | Raise ``maxAngle``, lower ``mergeDistance`` |
| Distant sounds are too dull | Raise ``lowpassMinFrequency`` in ``renderer.options`` |
| Distant sounds are too wet | Lower ``reverbMaxSend`` in ``renderer.options`` |

``renderer.options`` (lowpass, reverb, attenuation, smoothing) can also be changed at runtime, and applies on the next ``update()``.
