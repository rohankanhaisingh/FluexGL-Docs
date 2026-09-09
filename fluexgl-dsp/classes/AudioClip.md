# Class ``AudioClip``

A single decoded audio clip that can send its signal to a [``Channel``](./Channel.md) or a [``Master``](./Master.md), be played, looped, analyzed, and controlled over time.

## Example

```ts
import { AudioClip } from "@fluex/fluexgl-dsp";

const clip = new AudioClip(data);
clip.send(myChannel);

// Basic settings
clip.setVolume(0.8);
clip.setPanLevel(-0.2);
clip.setLoop(true);

// Start playback
clip.play();

// Manually seek to 30 seconds
clip.seek(30);
```

- - -

## Constructor
Constructs a new AudioClip from pre-decoded source data.

```ts
new AudioClip(data: AudioSourceData): AudioClip;
```

### Arguments
- ``data``: [``AudioSourceData``](../interfaces/AudioSourceData.md) - A typed object created when calling [``loadAudioSource()``](../helpers/LoadAudioSource.md) or [``loadAudioSourceFromBlob()``](../helpers/LoadAudioSourceFromBlob.md).

- - -

## Properties

### ``id: string``
A unique id, automatically generated when constructing a new audio clip. Should NOT be changed.

### ``label: string | null``
A custom label. Can be changed.

### ``loop: boolean``
Whether the clip is set to loop when played.

### ``isPlaying: boolean``
Indicates whether this clip is currently playing.

### ``startTime: number``
The ``AudioContext.currentTime`` at which the current playback started.

### ``offsetAtStart: number``
The offset (in seconds) inside the buffer from which playback started.

### ``playbackRate: number``
The current playback rate applied to buffer sources. Defaults to ``1``. Updated automatically by ``setPitch()``.

### ``pitch: number``
The current pitch offset in semitones. Defaults to ``0``. Updated by ``setPitch()``/``resetPitch()``.

### ``minPitchSemitones: number``
The minimum allowed pitch, in semitones, accepted by ``setPitch()``. Defaults to ``-24``.

### ``maxPitchSemitones: number``
The maximum allowed pitch, in semitones, accepted by ``setPitch()``. Defaults to ``24``.

### ``progressUpdateSpeed: number``
The interval in milliseconds used to track the audio clip's time progress. Default value is ``20``.

### ``gainNode: GainNode | null``
Per-clip gain node used to control volume.

### ``stereoPannerNode: StereoPannerNode | null``
Per-clip stereo panner node used to control pan level.

### ``context: AudioContext | null``
Audio context, usually inherited from the DSP's pipeline context.

### ``audioClipPlayer: AudioClipPlayer | null``
[``AudioClipPlayer``](./AudioClipPlayer.md) where this audio clip is attached to.

- - -

## Methods

### ``initialize(audioClipPlayer: AudioClipPlayer): void``
Initializes the audio clip using the [``AudioClipPlayer``](./AudioClipPlayer.md)'s audio context. Usually not needed because ``send(channel: Channel | Master)`` initializes it automatically.

#### Arguments
- ``audioClipPlayer``: [``AudioClipPlayer``](./AudioClipPlayer.md) AudioClipPlayer, which is usually automatically generated when creating a new [``Channel``](./Channel.md) or a new [``Master``](./Master.md);
  
#### Returns
- ``void``

### ``play(timestamp?: number, offset?: number): AudioClip | null``
Starts playback of the clip. The audio clip starts from the beginning if no arguments are provided.

#### Arguments
- ``timestamp?``: ``number`` - Absolute ``AudioContext.currentTime`` at which to start. If omitted, playback starts immediately. This argument is optional.
- ``offset?``: ``number`` - Offset in seconds inside the buffer to start from.
If omitted, uses the current ``offsetAtStart``.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.
- ``null`` - Returns ``null`` if this method failed, or if the maximum number of concurrent buffer source nodes (``maxAudioBufferSourceNodes``) has already been reached.

### ``seek(seconds: number): AudioClip | void``
Seeks to a given position (in seconds) inside the clip, clamped between ``0`` and ``duration``. If the clip is currently playing, playback is stopped and restarted from the new position; otherwise ``offsetAtStart`` is updated for the next ``play()`` call.

#### Arguments
- ``seconds``: ``number`` - The position, in seconds, to seek to.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``stop(): AudioClip | null``
Stops playback of this clip and disconnects all active buffer sources.

#### Arguments
No arguments

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.
- ``null`` - Returns ``null`` if this method failed.

### ``setVolume(volume: number): AudioClip``
Sets the clip volume using its ``GainNode``.

#### Arguments
- ``volume``: ``number`` - The desired gain value (linear)

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``setPanLevel(panLevel: number): AudioClip``
Sets the stereo pan level of the clip. Must be between ``-1`` and ``1``.

#### Arguments
- ``panLevel``: ``number`` - Pan value between -1 (full left) and 1 (full right).

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``setLoop(loop?: boolean): AudioClip``
Enables or disables looping of this clip.

#### Arguments
- ``loop?``: ``boolean`` - When omitted, defaults to true.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``setMaxAudioBufferSourceNodes(value: number): AudioClip``
Sets the maximum number of buffer source nodes (default ``1``). This can only be changed if the ``overrideMaxAudioBufferNodes`` property on ``DSP`` is set to ``true``; otherwise a warning is logged and the value is left unchanged.

#### Arguments
- ``value``: ``number`` - The desired maximum amount of buffer source nodes.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``disconnectAllAudioBufferSourceNodes(): boolean``
Stops and disconnects every currently active buffer source node for this clip.

#### Arguments
No arguments

#### Returns
- ``boolean`` - ``true`` if the operation ran, ``false`` if this clip has no ``context``.

### ``setPitch(semitones: number): AudioClip``
Sets the pitch of the clip in semitones, recalculating and applying the equivalent ``playbackRate`` (``2 ^ (semitones / 12)``) to all active buffer sources.

#### Arguments
- ``semitones``: ``number`` - The desired pitch offset in semitones. Must be between ``minPitchSemitones`` and ``maxPitchSemitones``.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``resetPitch(): AudioClip``
Resets the pitch back to ``0`` semitones. Shorthand for ``setPitch(0)``.

#### Arguments
No arguments

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``setPlaybackRateInSemitones(semitones: number): AudioClip``
> **Deprecated.** Use [``setPitch()``](#setpitchsemitones-number-audioclip) instead.

#### Arguments
- ``semitones``: ``number`` - The desired pitch offset in semitones.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``getChannelData(channel?: number): Float32Array``
Returns the raw PCM sample data for a single channel of the underlying ``AudioBuffer``.

#### Arguments
- ``channel?``: ``number`` - Zero-based channel index. Defaults to ``0``.

#### Returns
- ``Float32Array``

### ``addEventListener<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): () => void``
Registers a listener for clip events.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``once<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): () => void``
Registers a one-time event listener that automatically removes itself after the first call.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``removeEventListener<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): AudioClip``
Removes a specific listener from an event.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``AudioClip`` - The same instance, for chaining.

### ``clearEventListeners(event?: keyof AudioClipEventMap): AudioClip``
Clears event listeners.

#### Arguments
- ``event?: keyof AudioClipEventMap`` - If provided, clears listeners only for that event.

#### Returns
- ``AudioClip`` - The same instance, for chaining

### ``send(channel: Channel | Master): void``
Attaches this audio clip to a [``Channel``](./Channel.md) or a [``Master``](./Master.md) and sends its signal to the channel.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The channel where this audio clip's signal should be sent.

#### Returns
- ``void``

### ``unsend(channel: Channel | Master): void``
Detaches this audio clip from a [``Channel``](./Channel.md) or a [``Master``](./Master.md) and stops sending this clip's signal to the channel.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The channel that should stop receiving this clip's signal.

#### Returns
- ``void``

### ``detachFromAudioClipPlayer(audioClipPlayer: AudioClipPlayer): void``
Detaches this clip from a specific [``AudioClipPlayer``](./AudioClipPlayer.md). If that player was the clip's active player, the next remaining player (if any) becomes active. If no players remain, the clip is stopped.

#### Arguments
- ``audioClipPlayer``: [``AudioClipPlayer``](./AudioClipPlayer.md) - The player to detach from.

#### Returns
- ``void``

## Events

### ``"progress"``
Emitted periodically while the clip is playing, on an interval controlled by ``progressUpdateSpeed``.
Payload type ([``AudioClipOnProgressEvent``](../interfaces/AudioClipOnProgressEvent.md)):

```ts
type ProgressPayload = {
    current: number;          // Current playback position in seconds
    startTime: number;        // AudioContext timestamp when playback started
    offset: number;           // Offset in seconds at which playback started
    contextTimestamp: number; // Current AudioContext.currentTime
    formatted: string;        // Human readable time, e.g. "01:23"
};
```

Registered via
```ts
clip.addEventListener("progress", (event) => {
    console.log(event.current, event.formatted);
});
```

### ``"initialize"``
Emitted once, right after ``initialize()`` finishes wiring up this clip's audio nodes.
Payload (simplified):

```ts
type InitializePayload = {
    durationOfInitialization: number; // Time in ms the initialization took
    context: AudioContext | null;
};
```

### ``"play"``
Emitted every time ``play()`` successfully starts a new buffer source.
Payload (simplified):

```ts
type PlayPayload = {
    timestamp: number;                            // Date.now() at play time
    audioBufferSourceNodes: AudioBufferSourceNode[]; // All currently active buffer sources
    context: AudioContext;
};
```

- - -

## Getters and setters

### ``get currentPlaybackTime(): number``
Returns the current playback time in seconds relative to the start of the buffer. Returns ``0`` when the clip is not playing or no audio context is available. Otherwise: ``offsetAtStart`` + (``audioContext.currentTime`` - ``startTime``).

### ``get duration(): number``
Total duration of the underlying audio buffer in seconds.

### ``get volume(): number``
Current gain value read from ``gainNode``. Returns ``0`` when no gain node is available.

### ``get stereoPanning(): number``
Current pan value read from ``stereoPannerNode``. Returns ``1`` when no stereo panner node is available.

### ``get formattedDuration(): string``
Duration formatted as ``"mm:ss"``.

### ``get sampleRate(): number``
Sample rate of the underlying ``AudioBuffer``.

### ``get numberOfChannels(): number``
Number of channels in the underlying ``AudioBuffer``.

### ``get byteLength(): number``
Byte length of the original ArrayBuffer used to construct this clip.

- - -

## Examples

### Example 1: playing a simple background music
```ts
import { DspPipeline, loadAudioSource, AudioClip } from "@fluex/fluexgl-dsp";

(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet"
    });

    await pipeline.initializeDpsPipeline();
    
    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();
    if(!audioDevice) return;

    const audioSource = await loadAudioSource("/music.mp3");
    if(!audioSource) return;

    const audioClip = new AudioClip(audioSource);
    const myChannel = audioDevice.createChannel();

    audioClip.send(myChannel);
    myChannel.send(audioDevice.getMasterChannel());

    window.addEventListener("mousedown", function() {
        audioClip.play();
    });
})();
```

### Example 2: mimicking the sound of a machine gun
```ts
import { DspPipeline, loadAudioSource, AudioClip } from "@fluex/fluexgl-dsp";

(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.initializeDpsPipeline();
    
    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();
    if(!audioDevice) return;

    const audioSource = await loadAudioSource("/single-gun-shot.mp3");
    if(!audioSource) return;

    const audioClip = new AudioClip(audioSource);
    const myChannel = audioDevice.createChannel();

    audioClip.setMaxAudioBufferSourceNodes(200);
    audioClip.send(myChannel);
    myChannel.send(audioDevice.getMasterChannel());

    let lastTimestamp = Date.now();
    let isShooting = false;
    let shootDelayInMs = 100;

    function loop() {
        const now = Date.now();

        if((now - lastTimestamp >= shootDelayInMs) && isShooting) {
            audioClip.play();
            lastTimestamp = now;
        }

        return window.requestAnimationFrame(loop);
    }

    window.addEventListener("mousedown", function() { isShooting = true; });
    window.addEventListener("mouseup", function() { isShooting = false; });

    loop();
})();
```
