# IncomingProcessorMessage

Incoming messages from DSP processors.

```ts
interface IncomingProcessorMessage {
    id: string;
    timestamp: number;
    message: string;
    type: IncomingMessageType;
    processor: ProcessorData;
    additionalData?: any;
}
```

## About
Represents messages emitted by DSP processors.

## Properties
- `id`: `string` - Message identifier.
- `timestamp`: `number` - Time of emission.
- `message`: `string` - Message content.
- `type`: `IncomingMessageType` - Message type.
- `processor`: `ProcessorData` - Originating processor.
- `additionalData?`: `any` - Optional extra data.

