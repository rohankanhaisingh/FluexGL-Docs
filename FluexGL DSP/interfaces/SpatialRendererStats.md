# SpatialRendererStats

Counters returned by ``renderer.getStats()``.

```ts
interface SpatialRendererStats {
    sources: number;
    audible: number;
    virtual: number;
    voices: number;
    clusters: number;
    pooledVoices: number;
    suspendedLoops: number;
}
```

## About
Useful for debugging and for tuning ``maxVoices`` of a [``SpatialAudioRenderer``](../classes/SpatialAudioRenderer.md). Cheap enough to call every frame for a debug overlay.

## Properties
- `sources`: `number` - All sources of the renderer.
- `audible`: `number` - Sources loud enough to be heard.
- `virtual`: `number` - Audible sources without a voice, because the voice budget is used by louder sources.
- `voices`: `number` - Voices in use (individual and clusters).
- `clusters`: `number` - Voices shared by multiple sources.
- `pooledVoices`: `number` - Empty voices kept for reuse.
- `suspendedLoops`: `number` - Looping [``SoundInstance``](../classes/SoundInstance.md)s whose audio node is released because their source has no voice. See [Loop virtualization](../classes/SpatialAudioRenderer.md#loop-virtualization).

## Example
```ts
const { voices, virtual } = renderer.getStats();
debugText.textContent = `voices ${voices}/${renderer.options.maxVoices}, virtual ${virtual}`;
```
