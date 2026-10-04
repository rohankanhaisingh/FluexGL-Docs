# FluexGL DSP: guide for AI models and new developers

This page summarizes FluexGL DSP in one place: the mental model, the public API, the conventions and the pitfalls. It is written so that an AI model (or a developer new to the library) can write correct code without reading every page. Follow the links for details.

Package: ``@fluex/fluexgl-dsp``. Everything below is imported from the package root.

---

## 1. Mental model

FluexGL DSP is a channel-based audio engine on top of the Web Audio API. Most effects run as AudioWorklet processors backed by a Rust/WebAssembly module, so heavy processing happens on the audio thread.

```
DspPipeline        compiles the WASM module and prepares the worklet (once per page)
  -> AudioDevice   owns an AudioContext, a default Master, and creates Channels/Masters
       -> Master   final bus: input -> [effects] -> gain -> analyser -> speakers
       -> Channel  input -> [effects] -> stereo panner -> analyser -> gain -> output -> Master or another Channel
       -> AudioClip  a playable sound (decoded buffer), sent into Channels/Masters
       -> Effector   an effect in a Channel/Master chain (worklet or native nodes)
```

Spatial audio sits on top of this:

```
SpatialAudioRenderer2D / SpatialAudioRenderer3D   own Master + reverb Channel + Limiter + one listener
  -> SpatialAudioListener / SpatialAudioListener3D the "ears" (player or camera)
  -> SpatialAudioSource                             a positioned sound; AudioClips are attached to it
  -> SpatialAudioVoice (internal)                   lowpass -> panner -> master + reverb send; shared by clusters
```

---

## 2. Lifecycle (always in this order)

```ts
import { DspPipeline, AudioClip, loadAudioSource } from "@fluex/fluexgl-dsp";

// 1. After a user gesture (click/key), because of the browser's autoplay policy:
const pipeline = new DspPipeline({ pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm", pathToWorklet: "/bin/fluexgl-dsp-processor.worklet" });
await pipeline.initializeDpsPipeline();                         // note: "Dps", not "Dsp"
const audioDevice = await pipeline.resolveDefaultAudioOutputDevice(); // can be null
await audioDevice!.context.resume();

// 2. Load audio. Returns null on failure.
const data = await loadAudioSource("/audio/sound.ogg");

// 3. Route and play.
const channel = audioDevice!.createChannel("SFX");
channel.send(audioDevice!.getMasterChannel());
const clip = new AudioClip(data!);
clip.send(channel);
clip.play();
```

---

## 3. API quick reference

### Core
| API | Notes |
|---|---|
| ``new DspPipeline({ pathToWasm, pathToWorklet, options? })`` | ``options`` is a partial ``DspOptions``, e.g. ``{ overrideMaxAudioBufferNodes: true }``. Top-level keys replace the defaults, so pass a complete ``debugger`` object. |
| ``pipeline.initializeDpsPipeline(): Promise<boolean>`` | Alias ``init()``. Requests microphone permission to list devices. |
| ``pipeline.resolveDefaultAudioOutputDevice(): Promise<AudioDevice \| null>`` | Loads the worklet on the device's ``AudioContext``. Worklet effects only work on this context. |
| ``audioDevice.context`` | The ``AudioContext``. |
| ``audioDevice.getMasterChannel(): Master`` | Default master. |
| ``audioDevice.createChannel(label?): Channel`` | |
| ``audioDevice.createMasterChannel(): Master`` | An extra master, also connected to the speakers. |
| ``channel.send(target: Channel \| Master)`` / ``unsend(target)`` | Prevents feedback loops. |
| ``channel.addEffect(effect): Channel`` / ``removeEffect(effect)`` / ``moveEffectToIndex(effect, index \| "start" \| "end")`` | |
| ``master.attachEffect(effect)`` / ``detachEffect(effect)`` | Processed in attach order; no reordering. |
| ``channel.volume(v?)`` / ``channel.pan(v?)`` | See pitfalls: ``0`` is ignored. |
| ``loadAudioSource(path): Promise<AudioSourceData \| null>`` | Decodes the file. |
| ``new AudioClip(data)`` | Many clips can share one ``AudioSourceData``. |
| ``clip.send(channelOrMaster)`` / ``unsend(...)`` | A clip plays into every target it was sent to. |
| ``clip.play(when?, offset?)`` / ``stop()`` / ``seek(seconds)`` | ``play()`` returns ``null`` when the clip is already playing its maximum number of times. |
| ``clip.setLoop(bool)`` / ``setVolume(v)`` / ``setPanLevel(-1..1)`` / ``setPitch(semitones)`` | |
| ``clip.setMaxAudioBufferSourceNodes(n)`` | Overlapping playback. Requires ``overrideMaxAudioBufferNodes: true``. |
| ``clip.addEventListener("progress" \| "initialize" \| "play", cb)`` | |

### Effects
All effects extend ``Effector`` and are added with ``channel.addEffect()`` or ``master.attachEffect()``.

| Effect | Kind | Needs WASM |
|---|---|---|
| ``LowPassFilter``, ``HighPassFilter``, ``Chorus``, ``Reverb``, ``SoftClip``, ``HardClip`` | AudioWorklet | Yes |
| ``Compressor`` (``threshold``, ``knee``, ``ratio``, ``attack``, ``release``, ``makeupGain``) | Native ``DynamicsCompressorNode`` | No |
| ``Limiter`` (``ceiling``, ``release``, ``inputGain``) | Native ``DynamicsCompressorNode`` | No |
| ``Analyser`` | Native ``AnalyserNode`` | No |
| ``Distortion``, ``Equalizer``, ``Saturation``, ``StereoPanner``, ``MultibandCompressor``, delays | Placeholders, not functional yet | - |

Custom effects: extend ``Effector``, create nodes synchronously in ``initializeOnAttachment(context)``, and override the ``inputNode``/``outputNode`` getters (see [Effector](./classes/Effector.md)).

### Spatial audio
| API | Notes |
|---|---|
| ``new SpatialAudioRenderer2D(audioDeviceOrContext, options?)`` | StereoPanner. Pixels. ``refDistance 50``, ``maxDistance 1500``. |
| ``new SpatialAudioRenderer3D(audioDeviceOrContext, options?)`` | PannerNode, ``panningModel: "HRTF"`` by default. Meters. ``refDistance 1``, ``maxDistance 100``. |
| ``renderer.update()`` | Call every frame after moving the listener and sources. Or ``renderer.start()`` / ``stop()``. |
| ``renderer.createSource({ position, volume?, refDistance?, maxDistance?, clusterable?, reverbSendFactor?, airAbsorption? })`` | Returns a ``SpatialAudioSource``. |
| ``renderer.removeSource(source, dispose = true)`` | Fades out, then stops clips and releases nodes. |
| ``renderer.listener`` | ``SpatialAudioListener`` (2D) or ``SpatialAudioListener3D`` (3D). |
| ``renderer.master`` / ``renderer.limiter`` / ``renderer.reverbChannel`` | Own master, built-in limiter (``limiter: false`` disables), reverb bus. |
| ``renderer.setReverbEffect(effect \| null)`` | Default reverb is attached automatically once WASM is ready. |
| ``renderer.clustering`` / ``renderer.options`` | Mutable at runtime. |
| ``renderer.getClusters()`` | Debug info per voice. |
| ``(renderer as SpatialAudioRenderer3D).setPanningModel("HRTF" \| "equalpower" \| "stereo")`` | 3D only. |
| ``source.attachAudioClip(clip)`` | Then ``clip.play()``. Do NOT also ``clip.send()`` it to a channel. |
| ``source.setPosition(x, y, z?)`` / ``setVolume(v)`` / ``setAttenuation({...})`` | ``z`` is ignored in 2D. |
| ``source.state`` / ``source.audible`` / ``source.voice`` | Read-only results of the last ``update()``. |
| 2D listener: ``setPosition(x, y)``, ``setRotation(rad)``, ``lookAt(x, y)`` | Rotation optional, clockwise, ``0`` = facing up on screen. |
| 3D listener: ``setPosition(x, y, z)``, ``setOrientation(forward, up?)``, ``setYawPitch(yaw, pitch)``, ``lookAt(x, y, z)`` | y up, -z forward (like three.js). |

---

## 4. Conventions

- **Coordinates 2D**: screen space, y down by default (``yAxis: "up"`` for math coordinates). Listener rotation ``0`` faces up on the screen, so screen left/right = speaker left/right. Positive rotation is clockwise.
- **Coordinates 3D**: right-handed, y up, -z forward, identical to Web Audio and three.js. A ``THREE.Vector3`` can be passed wherever a ``Vector3`` is expected.
- **Listener space** (``source.state.local``): ``x`` = right, ``y`` = up, ``z`` = forward. ``azimuth`` is ``0`` in front and positive to the right.
- **Units**: whatever your world uses. Tune ``refDistance``/``maxDistance``/``clustering`` to it. The 2D defaults assume pixels, the 3D defaults assume meters.
- **Decibels**: ``Compressor`` and ``Limiter`` take dB for levels (``threshold``, ``ceiling``, ``makeupGain``, ``inputGain``) and seconds for times.
- **Chaining**: most setters return the instance (``Channel.addEffect``, listener and source setters, renderer methods). Effect setters return ``boolean``.
- **Errors**: many methods log through the internal debugger instead of throwing. With ``debugger.breakOnError: true`` (default), ``Debug.error`` also throws.

---

## 5. Pitfalls

1. **No sound at all?** The ``AudioContext`` must be resumed after a user gesture: ``await audioDevice.context.resume()``.
2. **Worklet effects throw "WebAssembly has not been compiled yet"**: add them only after ``await pipeline.initializeDpsPipeline()``, on the device from ``resolveDefaultAudioOutputDevice()``.
3. **``new LowPassFilter()`` and ``new Chorus()`` throw**: they require an options object. Use ``new LowPassFilter({})``.
4. **``channel.volume(0)`` and ``channel.pan(0)`` do nothing**: a value of ``0`` is ignored. Use ``channel.gainNode.gain.value = 0`` or ``channel.stereoPannerNode.pan.value = 0``.
5. **A clip plays twice / in the wrong place**: a clip plays into every target it was sent or attached to. Use one ``AudioClip`` per destination (they can share ``AudioSourceData``).
6. **``clip.play()`` returns ``null`` for rapid-fire sounds**: enable ``overrideMaxAudioBufferNodes`` on the pipeline and call ``clip.setMaxAudioBufferSourceNodes(n)``.
7. **Spatial sounds do not move**: ``renderer.update()`` must be called every frame (or ``renderer.start()`` once).
8. **2D sounds are panned the wrong way**: check ``yAxis`` and the rotation convention (``0`` = facing up on screen, clockwise).
9. **Side-scroller: sounds below the player are muffled**: set ``rearLowpassFactor: 1``.
10. **Effects attached to ``renderer.master`` come after the limiter**: create the renderer with ``limiter: false`` and attach your own ``Limiter`` last if you need effects before it.
11. **``renderer.listener`` is not ``AudioContext.listener``**: the renderers never use the Web Audio listener; do not set it.
12. **Disabling logs hides errors too**: ``options.debugger`` replaces the whole default object. Always pass all four fields: ``{ showInfo, showWarnings, showErrors, breakOnError }``.
13. **``NotchFilter`` cannot be imported**: the class exists in the source, but is not exported from the package root yet.

---

## 6. Where to look

| Task | Page |
|---|---|
| Install and initialize | [Getting started](../Getting%20started/How%20to%20use%20FluexGL-DSP.md) |
| Spatial audio concepts | [Spatial audio](../Getting%20started/Spatial%20audio.md) |
| Complete examples | [Examples](./examples/README.md) |
| Class reference | [classes/](./classes/README.md) |
| Effect reference | ``effects/<Name>.md`` |
| Option interfaces and types | ``interfaces/<Name>.md`` |
