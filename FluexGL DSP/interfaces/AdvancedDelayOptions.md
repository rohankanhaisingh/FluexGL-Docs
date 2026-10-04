# AdvancedDelayOptions

AdvancedDelay configuration.

```ts
interface AdvancedDelayOptions extends StereoDelayOptions {
    crossFeedback: number;
    lowCut: number;
    highCut: number;
    modulationRate: number;
    modulationDepth: number;
    drive: number;
}
```

## About
Used by [``AdvancedDelay``](../effects/AdvancedDelay.md). Also has every field of [``StereoDelayOptions``](./StereoDelayOptions.md).

## Properties
- `crossFeedback`: `number` - Between ``0`` (stereo) and ``1`` (ping-pong). Default ``0.2``.
- `lowCut`: `number` - Highpass in the feedback path (Hz), ``0`` (off) to ``2000``. Default ``120``.
- `highCut`: `number` - Lowpass in the feedback path (Hz), ``0`` = off. Default ``6000``.
- `modulationRate`: `number` - LFO speed (Hz), ``0`` to ``10``. Default ``0.5``.
- `modulationDepth`: `number` - LFO depth (ms), ``0`` (off) to ``20``. Default ``0``.
- `drive`: `number` - Saturation in the feedback path, ``0`` to ``1``. Default ``0.2``.
- Defaults of the inherited fields: ``delayLeftMs: 375``, ``delayRightMs: 500``, ``feedback: 0.45``, ``mix: 0.35``.
