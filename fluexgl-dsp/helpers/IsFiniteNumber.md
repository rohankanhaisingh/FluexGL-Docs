# IsFiniteNumber

Utility type guard for validating finite numeric values.

```ts
function isFiniteNumber(value: unknown): value is number;
```

> **Note:** this function is defined and exported from ``utilities/helpers.ts``, but it is **not** re-exported from the package's top-level ``index.ts``. It is not currently reachable as ``import { isFiniteNumber } from "@fluex/fluexgl-dsp"`` — it is used internally by [``coerceFiniteNumber()``](./CoerceFiniteNumber.md). Documented here for completeness.

- - -

## About
The `isFiniteNumber()` function is a small utility helper that checks whether a given value is a finite JavaScript number.

It performs two validations:
- Ensures the value is of type `number`
- Ensures the number is finite (not `NaN`, `Infinity`, or `-Infinity`)

Because it is implemented as a TypeScript type guard, this function also provides compile-time narrowing when used in conditional statements.

This helper is commonly used for validating DSP parameters and preventing unstable numeric values from propagating into audio processing logic.

## Parameters
- `value`: `unknown` – The value to be validated.

## Returns

- `boolean` –
  - `true` if the value is a finite number
  - `false` otherwise

When returning `true`, TypeScript will narrow `value` to type `number`.

## Error and warnings

This function does not emit any errors or warnings.
