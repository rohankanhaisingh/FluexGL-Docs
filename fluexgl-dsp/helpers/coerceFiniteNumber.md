# coerceFiniteNumber

Coerces an unknown value into a finite number, falling back to a default when validation fails.

```ts
function coerceFiniteNumber(value: unknown, fallback: number): number;
```

> **Note:** this function is defined and exported from ``utilities/helpers.ts``, but it is **not** re-exported from the package's top-level ``index.ts``. It is not currently reachable as ``import { coerceFiniteNumber } from "@fluex/fluexgl-dsp"`` — it is used internally by the effect classes (e.g. [``Chorus``](../effects/Chorus.md), [``HardClip``](../effects/HardClip.md)) to sanitize constructor and setter arguments. Documented here for completeness.

- - -

## About
The `coerceFiniteNumber()` function ensures that a usable finite numeric value is always returned.

It internally validates the provided value using `isFiniteNumber()` and:
- Returns the value directly if it is a finite number
- Returns the provided fallback value otherwise

This helper is especially useful for sanitizing external input, user-provided parameters, or messages received from AudioWorklet nodes before applying them to DSP logic.

## Parameters
- `value`: `unknown` – The value to validate and coerce.
- `fallback`: `number` – The fallback value to return if validation fails.

## Returns

- `number` – A guaranteed finite numeric value.

## Error and warnings

This function does not emit any errors or warnings.
