# ReverbOptions

Reverb effect configuration.

```ts
interface ReverbOptions {
    roomSize: number;
    damping: number;
    mix: number;
    stereoSpreadMs: number;
    strictMode: StrictMode;
}
```

## About
Defines parameters for the [``Reverb``](../effects/Reverb.md) DSP effect. Unlike the other effect option interfaces, ``ReverbOptions`` is declared directly alongside the ``Reverb`` class rather than in ``typings.ts``, but it is re-exported from the package root the same way.

## Properties
- `roomSize`: `number` - Simulated room size.
- `damping`: `number` - High-frequency damping amount.
- `mix`: `number` - Dry/wet mix ratio.
- `stereoSpreadMs`: `number` - Stereo spread, in milliseconds.
- `strictMode`: `StrictMode` - Validation behavior.
