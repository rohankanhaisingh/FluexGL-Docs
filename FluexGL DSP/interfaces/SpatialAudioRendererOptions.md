# SpatialAudioRendererOptions

Configuration shared by the 2D and 3D spatial renderers.

```ts
interface SpatialAudioRendererOptions extends SpatialAttenuationOptions {
    label: string | null;
    smoothing: number;
    crossfadeTime: number;
    lowpassMaxFrequency: number;
    lowpassMinFrequency: number;
    rearLowpassFactor: number;
    reverbMinSend: number;
    reverbMaxSend: number;
    silenceThreshold: number;
    maxVoices: number;
    clustering: Partial<SpatialClusteringOptions>;
    limiter: boolean | Partial<LimiterOptions>;
}
```

## About
Base options of [``SpatialAudioRenderer``](../classes/SpatialAudioRenderer.md). Extended by [``SpatialAudioRenderer2DOptions``](./SpatialAudioRenderer2DOptions.md) and [``SpatialAudioRenderer3DOptions``](./SpatialAudioRenderer3DOptions.md). Every field is optional.

The renderer maps the normalized distance (``0`` at ``refDistance``, ``1`` at ``maxDistance``) to:
- **Lowpass cutoff**: logarithmically from ``lowpassMaxFrequency`` to ``lowpassMinFrequency``, multiplied by up to ``rearLowpassFactor`` for sources behind the listener.
- **Reverb send**: from ``reverbMinSend`` to ``reverbMaxSend``, following a square root curve, multiplied by the source's ``reverbSendFactor``.

## Properties
- `label`: `string | null` - Custom label. Default ``null``.
- `distanceModel`, `refDistance`, `maxDistance`, `rolloffFactor` - Default attenuation for all sources. See [``SpatialAttenuationOptions``](./SpatialAttenuationOptions.md).
- `smoothing`: `number` - Time constant (seconds) used to smooth parameter changes. Default ``0.05``.
- `crossfadeTime`: `number` - Crossfade time (seconds) when a source moves between voices. Default ``0.12``.
- `lowpassMaxFrequency`: `number` - Lowpass cutoff (Hz) at ``refDistance``. Default ``20000``.
- `lowpassMinFrequency`: `number` - Lowpass cutoff (Hz) at ``maxDistance``. Default ``600``.
- `rearLowpassFactor`: `number` - Extra cutoff multiplier for sources directly behind the listener. ``1`` disables it. Default ``0.6``.
- `reverbMinSend`: `number` - Reverb send at ``refDistance``. Default ``0.05``.
- `reverbMaxSend`: `number` - Reverb send at ``maxDistance``. Default ``0.6``.
- `silenceThreshold`: `number` - Gain below which a source is inaudible and gets no voice. It becomes audible again at twice this value. Default ``0.001``.
- `maxVoices`: `number` - Maximum number of voices (a cluster counts as one). Above it, the quietest sources become virtual. Default ``64`` (2D) or ``32`` (3D). See [Voice budget](../classes/SpatialAudioRenderer.md#voice-budget).
- `clustering`: [`Partial<SpatialClusteringOptions>`](./SpatialClusteringOptions.md) - Clustering configuration.
- `limiter`: `boolean` | [`Partial<LimiterOptions>`](./LimiterOptions.md) - Safety [``Limiter``](../effects/Limiter.md) on the renderer's master channel. ``false`` disables it, an object configures it. Default ``true``.
