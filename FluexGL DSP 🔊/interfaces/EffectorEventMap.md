# EffectorEventMap

Effector-related event map.

```ts
interface EffectorEventMap {
    "incoming-processor-message": (message: IncomingProcessorMessage) => void;
    "incoming-processor-warning": (message: IncomingProcessorMessage) => void;
    "incoming-processor-error": (message: IncomingProcessorMessage) => void;
    "initialized-on-channel-attachment": (message: IncomingProcessorMessage) => void;
    "processor-wasm-instantiated": (message: IncomingProcessorMessage) => void;
}
```

## About
Maps processor lifecycle and message events.

## Properties
- `incoming-processor-message`: Generic processor message event.
- `incoming-processor-warning`: Processor warning event.
- `incoming-processor-error`: Processor error event.
- `initialized-on-channel-attachment`: Fired when attached to a channel.
- `processor-wasm-instantiated`: Fired when WASM is instantiated.

