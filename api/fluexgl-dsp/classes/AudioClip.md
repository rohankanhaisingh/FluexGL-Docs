# Class ``AudioClip``

A single decoded audio clip that can send its signal to a [``Channel``](./Channel.md) or a [``Master``](./Master.md), be played, looped, analyzed, and controlled over time.

## Example

```ts
import { AudioClip } from "@fluex/fluexgl-dsp";

const clip = new AudioClip(data);
clip.Send(myChannel);

// Basic settings
clip.SetVolume(0.8);
clip.SetPanLevel(-0.2);
clip.Loop(true);

// Start playback
clip.Play();

// Manually seek to 30 seconds
clip.Seek(30);
```

- - -

## Constructor
Constructs a new AudioClip from pre-decoded source data.

```ts
new AudioClip(data: AudioSourceData): AudioClip;
```

### Arguments
- ``data``: [``AudioSourceData``](../interfaces/AudioSourceData.md) - A typed object created when calling [``LoadAudioSource()``](../helpers/LoadAudioSource.md) or [``LoadAudioSourceFromBlob()``](../helpers/LoadAudioSourceFromBlob.md).

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

### ``Initialize(audioClipPlayer: AudioClipPlayer): void;``
Initializes the audio clip using the [``AudioClipPlayer``](./AudioClipPlayer.md)'s audio context. Usually not needed because ``Send(channel: Channel | Master)`` initializes it automatically.

#### Arguments
- ``audioClipPlayer``: [``AudioClipPlayer``](./AudioClipPlayer.md) AudioClipPlayer, which is usually automatically generated when creating a new [``Channel``](./Channel.md) or a new [``Master``](./Master.md);
  
#### Returns
- ``void``

### ``Play(timestamp?: number, offset?: number): AudioClip | null``
Starts playback of the clip. The audio clip starts from the beginning if no arguments are provided.

#### Arguments
- ``timestamp?``: ``number`` - Absolute ``AudioContext.currentTime`` at which to start. If omitted, playback starts immediately. This argument is optional.
- ``offset?``: ``number`` - Offset in seconds inside the buffer to start from.
If omitted, uses the current ``offsetAtStart``.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.
- ``null`` - Returns ``null`` if this method failed.

### ``Stop(): AudioClip | null``
Stops playback of this clip and disconnects all active buffer sources.

#### Arguments
No arguments

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.
- ``null`` - Returns ``null`` if this method failed.

### ``SetVolume(volume: number): AudioClip``
Sets the clip volume using its ``GainNode``.

#### Arguments
- ``volume``: ``number`` - The desired gain value (linear)

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``SetPanLevel(panLevel: number): AudioClip``
Sets the stereo pan level of the clip.

#### Arguments
- ``panLevel``: ``number`` - Pan value between -1 (full left) and 1 (full right).

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``Loop(loop?: boolean): AudioClip``
Enables or disables looping of this clip.

#### Arguments
- ``loop?``: ``boolean`` - When omitted, defaults to true.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``SetMaxAudioBufferSourceNodes(value: number): AudioClip``
Sets the maximum number of buffer source nodes (default ``1``). This can only be changed if the ``overrideMaxAudioBufferNodes`` property on ``DSP`` is set to ``true``.

#### Arguments
- ``value``: ``number`` - The desired maximum amount of buffer source nodes.

#### Returns
- ``AudioClip`` - The same AudioClip. Can be used to stack methods.

### ``AddEventListener<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): () => void``
Registers a listener for clip events.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``Once<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): () => void``
Registers a one-time event listener that automatically removes itself after the first call.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``() => void`` - Unsubscribe function to remove the listener.

### ``RemoveEventListener<K extends keyof AudioClipEventMap>(event: K, cb: AudioClipEventMap[K]): AudioClip``
Removes a specific listener from an event.

#### Arguments
- ``event: K (keyof AudioClipEventMap)`` - Event name (e.g. "progress").
- ``cb: (event: (event from keyof AudioClipEventMap)) => void`` - Callback function.

#### Returns
- ``AudioClip`` - The same instance, for chaining.

### ``ClearEventListeners(event?: keyof AudioClipEventMap): AudioClip``
Clears event listeners.

#### Arguments
- ``event?: keyof AudioClipEventMap`` - If provided, clears listeners only for that event.

#### Returns
- ``AudioClip`` - The same instance, for chaining

### ``Send(channel: Channel | Master): void``
Attaches this audio clip to a [``Channel``](./Channel.md) or a [``Master``](./Master.md) and sends its signal to the channel.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The channel where this audio clip's signal should be sent.

#### Returns
- ``void``

### ``Unsend(channel: Channel | Master): void``
Detaches this audio clip from a [``Channel``](./Channel.md) or a [``Master``](./Master.md) and stops sending this clip's signal to the channel.

#### Arguments
- ``channel``: [``Channel``](./Channel.md) | [``Master``](./Master.md) - The channel that should stop receiving this clip's signal.

#### Returns
- ``void``

## Events

### ``"progress"``
Emitted periodically while the clip is playing.
Payload type (simplified):

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
clip.AddEventListener("progress", (event) => {
    console.log(event.current, event.formatted);
});
```

- - -

## Getters and setters

### ``get currentPlaybackTime(): number``
Returns the current playback time in seconds relative to the start of the buffer. Returns ``0`` when the clip is not playing or no audio context is available. Otherwise: ``offsetAtStart`` + (``audioContext.currentTime`` - ``startTime``).

### ``get duration(): number``
Total duration of the underlying audio buffer in seconds.

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
import { DspPipeline, LoadAudioSource, AudioClip } from "@fluex/fluexgl-dsp";

(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet"
    });

    await pipeline.InitializeDspPipeline();
    
    const audioDevice = await pipeline.ResolveDefaultAudioOutputDevice();
    if(!audioDevice) return;

    const audioSource = await LoadAudioSource("/music.mp3");
    if(!audioSource) return;

    const audioClip = new AudioClip(audioSource);
    const myChannel = audioDevice.CreateChannel();

    audioClip.Send(myChannel);
    myChannel.Send(audioDevice.GetMasterChannel());

    window.addEventListener("mousedown", function() {
        audioClip.Play();
    });
})();
```

### Example 2: mimicking the sound of a machine gun
```ts
import { DspPipeline, LoadAudioSource, AudioClip } from "@fluex/fluexgl-dsp";

(async function() {
    const pipeline = new DspPipeline({
        pathToWasm: "/data/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/data/fluexgl-dsp-processor.worklet",
        options: {
            overrideMaxAudioBufferNodes: true
        }
    });

    await pipeline.InitializeDspPipeline();
    
    const audioDevice = await pipeline.ResolveDefaultAudioOutputDevice();
    if(!audioDevice) return;

    const audioSource = await LoadAudioSource("/single-gun-shot.mp3");
    if(!audioSource) return;

    const audioClip = new AudioClip(audioSource);
    const myChannel = audioDevice.CreateChannel();

    audioClip.SetMaxAudioBufferSourceNodes(200);
    audioClip.Send(myChannel);
    myChannel.Send(audioDevice.GetMasterChannel());

    let lastTimestamp = Date.now();
    let isShooting = false;
    let shootDelayInMs = 100;

    function loop() {
        const now = Date.now();

        if((now - lastTimestamp >= shootDelayInMs) && isShooting) {
            audioClip.Play();
            lastTimestamp = now;
        }

        return window.requestAnimationFrame(loop);
    }

    window.addEventListener("mousedown", function() { isShooting = true; });
    window.addEventListener("mouseup", function() { isShooting = false; });

    loop();
})();
```
