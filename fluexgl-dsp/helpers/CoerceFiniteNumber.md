# CoerceFiniteNumber

Coerces an unknown value into a finite number, falling back to a default when validation fails.

```ts
function CoerceFiniteNumber(value: unknown, fallback: number): number;
```

- - -

## About
The `CoerceFiniteNumber()` function ensures that a usable finite numeric value is always returned.

It internally validates the provided value using `IsFiniteNumber()` and:
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
