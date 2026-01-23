# ProcessorData

DSP processor metadata.

```ts
interface ProcessorData {
    id: string | ProcessorIdentificationCodes.UnknownProcessorId;
    name: string | ProcessorIdentificationCodes.UnknownProcessorName;
    createdAt: number | ProcessorIdentificationCodes.UnknownProcessorCreationDate;
}
```

## About
Represents metadata about a DSP processor.

## Properties
- `id`: `string | UnknownProcessorId` - Processor identifier.
- `name`: `string | UnknownProcessorName` - Processor name.
- `createdAt`: `number | UnknownProcessorCreationDate` - Creation timestamp.

