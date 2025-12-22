# AudioClipOnProgressEvent

Describes audio clip progress events.

```ts
interface AudioClipOnProgressEvent {
    startTime: number;
    offset: number;
    current: number;
    contextTimestamp: number;
    formatted: string;
}
```

## About
Represents playback progress state.

## Properties
- `startTime`: `number` - Playback start time.
- `offset`: `number` - Offset from the start.
- `current`: `number` - Current playback position.
- `contextTimestamp`: `number` - Audio context timestamp.
- `formatted`: `string` - Human-readable formatted time.

