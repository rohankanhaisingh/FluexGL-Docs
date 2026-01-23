# AudioClipEventMap

Maps audio clip events to handlers.

```ts
interface AudioClipEventMap {
    "progress": (event: AudioClipOnProgressEvent) => void;
}
```

## About
Maps audio clip events to their handlers.

## Properties
- `progress`: `(event: AudioClipOnProgressEvent) => void` - Fired during playback progression.

