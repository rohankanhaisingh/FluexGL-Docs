# InputChannelOptions

Capture options of an InputChannel.

```ts
interface InputChannelOptions {
    echoCancellation: boolean;
    noiseSuppression: boolean;
    autoGainControl: boolean;
    channelCount: number | null;
    fallbackToDefault: boolean;
}
```

## About
Used by [``InputChannel``](../classes/InputChannel.md) and [``AudioDevice.createInputChannel()``](../classes/AudioDevice.md). The first four options are passed to ``getUserMedia()`` as constraints, so they are applied the next time a device is opened. They have no effect on streams passed to ``setMediaStream()``.

The browser's voice processing is disabled by default, so effects receive the unprocessed signal. That is what you want for music, instruments and sound design. For voice chat, enable ``echoCancellation``, ``noiseSuppression`` and ``autoGainControl``.

## Properties
- `echoCancellation`: `boolean` - Browser echo cancellation. Default ``false``.
- `noiseSuppression`: `boolean` - Browser noise suppression. Default ``false``.
- `autoGainControl`: `boolean` - Browser automatic gain control. Default ``false``.
- `channelCount`: `number | null` - Preferred amount of input channels (``1`` = mono, ``2`` = stereo), passed as an ``ideal`` constraint. ``null`` lets the browser decide. Default ``null``.
- `fallbackToDefault`: `boolean` - Whether the default input device is opened when the current input device is disconnected. Never applies to external streams. Default ``true``.
