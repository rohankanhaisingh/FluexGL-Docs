# LimiterOptions

Limiter configuration.

```ts
interface LimiterOptions {
    ceiling: number;
    release: number;
    inputGain: number;
}
```

## About
Defines parameters for the [``Limiter``](../effects/Limiter.md) effect, and for the ``limiter`` option of [``SpatialAudioRendererOptions``](./SpatialAudioRendererOptions.md). Values outside their range are clamped.

## Properties
- `ceiling`: `number` - Maximum output level (dB). Between ``-24`` and ``0``. Default ``-1``.
- `release`: `number` - Time (seconds) for the limiter to recover. Between ``0.01`` and ``1``. Default ``0.1``.
- `inputGain`: `number` - Gain (dB) applied before limiting. Between ``-24`` and ``24``. Default ``0``.
