# StrictMode

Validation mode flag used by most DSP effect options (e.g. [``ChorusEffectOptions``](./ChorusEffectOptions.md), [``HardClipOptions``](./HardClipOptions.md), [``SoftClipOptions``](./SoftClipOptions.md), [``LowPassFilterOptions``](./LowPassFilterOptions.md)).

```ts
enum StrictMode {
    Disabled = 0x00,
    Enabled = 0x01
}
```

- - -

## About
``StrictMode`` is a numeric enum exported directly from the package root (``import { StrictMode } from "@fluex/fluexgl-dsp"``). Every effector that accepts a ``strictMode`` option coerces whatever value it receives to either ``StrictMode.Enabled`` or ``StrictMode.Disabled`` (falling back to ``StrictMode.Disabled`` for anything else), and forwards it to the underlying AudioWorklet processor as part of its options object.

## Members
- `Disabled = 0x00` - Default. Relaxed validation on the processor side.
- `Enabled = 0x01` - Stricter validation on the processor side.

<!-- TODO: verify -- the exact validation behavior difference between Disabled and Enabled lives inside the compiled WebAssembly/AudioWorklet processor code, which is outside this repository's lib/src TypeScript sources, so it isn't documented here in detail. -->
