# AudioSourceData

Describes loaded audio source data.

```ts
interface AudioSourceData {
    arrayBuffer: ArrayBuffer;
    audioBuffer: AudioBuffer;
    id: string;
    timestamp: number;
}
```

## About
Represents decoded and raw audio source data.

## Properties
- `arrayBuffer`: `ArrayBuffer` - Raw binary audio data.
- `audioBuffer`: `AudioBuffer` - Decoded audio buffer.
- `id`: `string` - Unique identifier of the audio source.
- `timestamp`: `number` - Creation timestamp.

