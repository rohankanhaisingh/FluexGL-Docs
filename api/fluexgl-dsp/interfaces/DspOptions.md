# DspOptions

Global configuration options for the DSP system.

```ts
interface DspOptions {
    maxMasterChannels: number;
    maxTotalChannels: number;
    sampleRate: number;
    spatialization: ChannelSpatialization;
    debugger: DspDebuggerOptions;
    overrideMaxAudioBufferNodes: boolean;
}
```

## About
Specifies global DSP behavior, limits, and runtime configuration.

## Properties
- `maxMasterChannels`: `number` - Maximum number of master output channels.
- `maxTotalChannels`: `number` - Maximum number of total audio channels.
- `sampleRate`: `number` - Audio sample rate used by the DSP pipeline.
- `spatialization`: `ChannelSpatialization` - Defines how channels are spatially processed.
- `debugger`: `DspDebuggerOptions` - Debugging configuration.
- `overrideMaxAudioBufferNodes`: `boolean` - Overrides browser-imposed audio buffer node limits.

