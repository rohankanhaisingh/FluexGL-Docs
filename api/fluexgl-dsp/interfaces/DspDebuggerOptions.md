# DspDebuggerOptions

Configuration options for the DSP debugger.

```ts
interface DspDebuggerOptions {
    breakOnError: boolean;
    showInfo: boolean;
    showErrors: boolean;
    showWarnings: boolean;
}
```

## About
Defines how the DSP debugging system behaves during runtime.

## Properties
- `breakOnError`: `boolean` - Whether execution should pause when an error occurs.
- `showInfo`: `boolean` - Enable informational debug messages.
- `showErrors`: `boolean` - Enable error logging.
- `showWarnings`: `boolean` - Enable warning logging.

