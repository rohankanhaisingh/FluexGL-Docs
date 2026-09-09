# AudioClipEventMap

Maps audio clip events to handlers.

```ts
interface AudioClipEventMap {
    "progress": (event: AudioClipOnProgressEvent) => void;
    "initialize": (event: AudioClipOnInitializeEvent) => void;
    "play": (event: AudioClipOnPlayEvent) => void;
}
```

## About
Maps audio clip events to their handlers.

## Properties
- `progress`: `(event: AudioClipOnProgressEvent) => void` - Fired during playback progression. See [``AudioClipOnProgressEvent``](./AudioClipOnProgressEvent.md).
- `initialize`: `(event: AudioClipOnInitializeEvent) => void` - Fired once the clip has finished wiring up its audio nodes via ``initialize()``. Payload: `{ durationOfInitialization: number; context: AudioContext | null }`.
- `play`: `(event: AudioClipOnPlayEvent) => void` - Fired every time ``play()`` starts a new buffer source. Payload: `{ timestamp: number; audioBufferSourceNodes: AudioBufferSourceNode[]; context: AudioContext }`.

<!-- TODO: verify -- AudioClipOnInitializeEvent and AudioClipOnPlayEvent are defined in typings.ts but are not re-exported from the package's top-level index.ts (only AudioClipOnProgressEvent is), so they have no linkable interfaces page of their own yet. -->

