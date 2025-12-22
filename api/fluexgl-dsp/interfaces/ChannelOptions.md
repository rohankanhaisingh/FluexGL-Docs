# ChannelOptions

Configuration options for DSP channels.

```ts
interface ChannelOptions {
    label: string | null;
    maxAudioNodes: number;
    maxEffects: number;
}
```

## About
Defines limits and metadata for audio channels.

## Properties
- `label`: `string | null` - Optional human-readable channel label.
- `maxAudioNodes`: `number` - Maximum number of audio nodes allowed.
- `maxEffects`: `number` - Maximum number of effects allowed on the channel.

