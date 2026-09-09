# DefaultSampleRate

The fallback sample rate (in Hz) used before an effect has been attached to a real ``AudioContext``.

```ts
const DEFAULT_SAMPLE_RATE: number;
```

- - -

## About
``DEFAULT_SAMPLE_RATE`` is a constant equal to ``44100``. It is used, for example, as the initial ``contextSampleRate`` of [``HighPassFilter``](../effects/HighPassFilter.md) and [``NotchFilter``](../effects/NotchFilter.md) before ``initializeOnAttachment()`` has run.

## Value

```ts
44100
```

## Error and warnings

This constant does not emit any errors or warnings.
