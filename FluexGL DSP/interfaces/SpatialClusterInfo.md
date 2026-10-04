# SpatialClusterInfo

Debug information about one voice of a spatial renderer.

```ts
interface SpatialClusterInfo {
    voiceId: string;
    isCluster: boolean;
    sourceIds: string[];
    azimuth: number;
    elevation: number;
    distance: number;
}
```

## About
Returned by ``renderer.getClusters()``, one entry per active voice. Useful for visualizing clusters.

## Properties
- `voiceId`: `string` - Id of the [``SpatialAudioVoice``](../classes/SpatialAudioVoice.md).
- `isCluster`: `boolean` - Whether the voice is shared by multiple sources.
- `sourceIds`: `string[]` - Ids of the sources rendered by this voice.
- `azimuth`: `number` - Horizontal angle (radians) of the voice: the cluster centre, or the direction of its only source.
- `elevation`: `number` - Vertical angle (radians) of the voice.
- `distance`: `number` - Average distance of the members to the listener.
