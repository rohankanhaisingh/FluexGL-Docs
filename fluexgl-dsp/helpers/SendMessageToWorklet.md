# SendMessageToWorklet

Sends a typed command message to an AudioWorklet node with basic validation and safety checks.

```ts
function SendMessageToWorklet<T, K = any>(
    node: AudioWorkletNode | null,
    commandId: T,
    data: K
): boolean;
```

- - -

## About
The `SendMessageToWorklet()` function provides a safe and consistent way to send command messages to an `AudioWorkletNode`.

Before sending the message, the function:
- Verifies that the target worklet node exists
- Prevents non-finite numeric values (`NaN`, `Infinity`) from being sent
- Emits warnings when invalid data is detected

Messages are sent through the `MessagePort` associated with the `AudioWorkletNode` and follow a simple `{ commandId, data }` structure.

This function is commonly used for real-time DSP parameter updates, control messages, and state synchronization between the main thread and the AudioWorklet.

## Parameters
- `node`: `AudioWorkletNode | null` – The target AudioWorklet node. If `null`, the message will not be sent.
- `commandId`: `T` – A command identifier used by the AudioWorklet to route or interpret the message.
- `data`: `K` – Arbitrary payload data associated with the command.

## Returns

- `boolean` –
  - `true` if the message was successfully sent
  - `false` if the node was null or the data was rejected by validation

## Error and warnings

### Warning: non-finite numeric value
If `data` is a number and is not finite, the message is rejected and a warning is emitted:

> "Refusing to send non-finite numeric value to AudioWorkletNode."

This prevents unstable or undefined values from reaching the DSP layer.

No custom FluexGL-DSP error codes are emitted by this function.
