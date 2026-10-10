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
       -> Channel  input -> [effects] -> [stereo panner] -> [analyser] -> gain/output -> Master or another Channel
                   (panner and analyser only exist once used; use channels as buses, not one per game object)
       -> InputChannel  a Channel fed by a microphone or a MediaStream (WebRTC) instead of AudioClips
       -> AudioClip  a playable sound (decoded buffer), sent into Channels/Masters; for music and editing
       -> Sound      a game sound effect: one shared buffer, lightweight SoundInstances, instance limits
       -> Effector   an effect in a Channel/Master chain (worklet or native nodes)
```

Spatial audio sits on top of this:

```
SpatialAudioRenderer2D / SpatialAudioRenderer3D   own Master (or an `output` bus) + reverb Channel + Limiter + one listener
  -> SpatialAudioListener / SpatialAudioListener3D the "ears" (player or camera)
  -> SpatialAudioSource                             a positioned emitter (2 gain nodes), one per game object; plays Sounds,
                                                    AudioClips and Channels; `bus` routes it to a bus Channel
  -> renderer.playAt(sound, position)               fire-and-forget Sound on a pooled source (explosions, impacts)
  -> SpatialAudioVoice (internal)                   lowpass -> panner -> bus + reverb send; shared by clusters on the same bus
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
| ``new DspPipeline({ pathToWasm, pathToWorklet, options? })`` | ``options`` is a partial ``DspOptions``, e.g. ``{ overrideMaxAudioBufferNodes: true, debugger: { showInfo: false } }``. Nested objects are merged with the defaults. |
| ``pipeline.initializeDpsPipeline(): Promise<boolean>`` | Alias ``init()``. Requests microphone permission to list devices. |
| ``pipeline.resolveDefaultAudioOutputDevice(): Promise<AudioDevice \| null>`` | Loads the worklet on the device's ``AudioContext``. Worklet effects only work on this context. |
| ``audioDevice.context`` | The ``AudioContext``. |
| ``audioDevice.getMasterChannel(): Master`` | Default master. |
| ``audioDevice.createChannel(label?): Channel`` | |
| ``audioDevice.createMasterChannel(): Master`` | An extra master, also connected to the speakers. |
| ``audioDevice.setOutputDevice(deviceOrId \| null): Promise<boolean>`` | Switches the output device at runtime; the graph stays intact. Needs ``AudioDevice.supportsOutputDeviceSelection`` (Chromium). |
| ``audioDevice.createInputChannel(deviceOrId?, label?, options?): Promise<InputChannel>`` | Opens a microphone on a new channel. Not connected to anything by default. |
| ``audioDevice.addEventListener("output-device-changed" \| "output-device-lost" \| "devices-changed", cb)`` | Unplugged output devices fall back to the default device. |
| ``inputChannel.setInputDevice(deviceOrId \| null)`` / ``setMediaStream(stream)`` / ``setMuted(bool)`` / ``close()`` | Switch microphone, use a remote stream, mute, release. |
| ``listAudioInputDevices()`` / ``listAudioOutputDevices()`` / ``findDefaultAudioDevice(kind)`` / ``watchAudioDevices(cb)`` | Plain ``MediaDeviceInfo`` lists. The old ``resolveAudio*Devices`` helpers are deprecated. |
| ``channel.send(target: Channel \| Master)`` / ``unsend(target)`` | Prevents feedback loops. ``unsend`` returns ``false`` (no error) when there was no link. Sending to several targets splits, several channels sending to one target merges. |
| ``channel.isSentTo(target)`` / ``unsendFromAllMasters()`` / ``unsendFromAll()`` | ``channel.masters`` lists the masters a channel is attached to. |
| ``channel.addEffect(effect): Channel`` / ``removeEffect(effect)`` / ``moveEffectToIndex(effect, index \| "start" \| "end")`` | |
| ``master.attachEffect(effect)`` / ``detachEffect(effect)`` | Processed in attach order; no reordering. |
| ``channel.volume(v?)`` / ``channel.pan(v?)`` | Getter and setter in one. ``0`` is a valid value. The panner is created on the first non-zero ``pan()``. |
| ``channel.enableAnalyser(options?): AnalyserNode`` / ``disableAnalyser()`` | Channels have no analyser by default; ``channel.analyserNode`` is ``null`` until enabled. |
| ``channel.dispose()`` | Stops clips, removes outgoing links, releases nodes. |
| ``loadAudioSource(path): Promise<AudioSourceData \| null>`` | Decodes the file. |
| ``new AudioClip(data)`` | Many clips can share one ``AudioSourceData``. |
| ``clip.send(channelOrMaster)`` / ``unsend(...)`` | A clip plays into every target it was sent to. |
| ``clip.play(when?, offset?)`` / ``stop()`` / ``seek(seconds)`` | ``play()`` returns ``null`` when the clip is already playing its maximum number of times. |
| ``clip.setLoop(bool)`` / ``setVolume(v)`` / ``setPanLevel(-1..1)`` / ``setPitch(semitones)`` | |
| ``clip.setMaxAudioBufferSourceNodes(n)`` | Overlapping playback. Requires ``overrideMaxAudioBufferNodes: true``. |
| ``clip.addEventListener("progress" \| "initialize" \| "play", cb)`` | |
| ``Sound.load(audioDeviceOrContext, url, options?): Promise<Sound>`` | Game sound effects. Decodes on the given context, cached per url. Options: ``volume``, ``volumeVariation``, ``pitch``, ``pitchVariation``, ``maxInstances`` (8), ``steal`` (``"oldest"`` \| ``"quietest"`` \| ``"none"``), ``minInterval``, ``loop``. |
| ``sound.play(channelOrMaster, { volume?, pitch?, loop?, offset?, when? })`` | Not positioned (UI, the player's own sounds). Returns a ``SoundInstance`` or ``null`` when skipped. |
| ``instance.stop(fade?)`` / ``setVolume(v)`` / ``setPosition(x, y, z?)`` / ``onEnded(cb)`` / ``source`` | ``setPosition`` only for ``playAt()`` instances. An ended instance can not be controlled. |

### Effects
All effects extend ``Effector`` and are added with ``channel.addEffect()`` or ``master.attachEffect()``.

| Effect | Kind | Needs WASM |
|---|---|---|
| ``LowPassFilter``, ``HighPassFilter``, ``NotchFilter``, ``Chorus``, ``Reverb``, ``SoftClip``, ``HardClip`` | AudioWorklet | Yes |
| ``Equalizer`` (up to 8 bands: ``peaking``, ``lowshelf``, ``highshelf``, ``lowpass``, ``highpass``, ``notch``, ``bandpass``) | AudioWorklet | Yes |
| ``MonoDelay``, ``StereoDelay``, ``PingPongDelay``, ``AdvancedDelay`` (cross feedback, low/high cut, modulation, drive) | AudioWorklet, one shared delay engine | Yes |
| ``Saturation`` (``"soft"``, ``"tube"``, ``"tape"``, anti-aliased) | AudioWorklet | Yes |
| ``Compressor`` (``threshold``, ``knee``, ``ratio``, ``attack``, ``release``, ``makeupGain``) | Native ``DynamicsCompressorNode`` | No |
| ``MultibandCompressor`` (3 bands, Linkwitz-Riley crossovers) | Native | No |
| ``Limiter`` (``ceiling``, ``release``, ``inputGain``) | Native compressor + soft clipper | No |
| ``StereoPanner`` (``pan``, ``width``) | Native | No |
| ``StereoMono`` (``mode``: ``stereo``, ``mono``, ``swap``, ``left``, ``right``, ``left-to-both``, ``right-to-both``, ``mid``, ``side``; delay and polarity per side). ``StereoMono.split(channel, "left-right" \| "mid-side")`` | Native | No |
| ``Analyser`` | Native ``AnalyserNode`` | No |
| ``Distortion`` | Placeholder, not functional yet | - |

The new WASM effects (``Equalizer``, the delays, ``Saturation``) require a worklet/WASM build from FluexGL-DSP-WebAssembly 0.4.9 or newer.

Custom effects: extend ``Effector``, create nodes synchronously in ``initializeOnAttachment(context)``, and override the ``inputNode``/``outputNode`` getters (see [Effector](./classes/Effector.md)).

### Spatial audio
| API | Notes |
|---|---|
| ``new SpatialAudioRenderer2D(audioDeviceOrContext, options?)`` | StereoPanner. Pixels. ``refDistance 50``, ``maxDistance 1500``. |
| ``new SpatialAudioRenderer3D(audioDeviceOrContext, options?)`` | PannerNode, ``panningModel: "HRTF"`` by default. Meters. ``refDistance 1``, ``maxDistance 100``. |
| ``renderer.update()`` | Call every frame after moving the listener and sources. Or ``renderer.start()`` / ``stop()``. |
| ``renderer.createSource({ position, volume?, refDistance?, maxDistance?, clusterable?, reverbSendFactor?, airAbsorption? })`` | Returns a ``SpatialAudioSource``. |
| ``renderer.removeSource(source, dispose = true)`` | Fades out, then stops sounds and clips and releases nodes. |
| ``renderer.playAt(sound, position, { bus?, loop?, volume?, cull?, refDistance?, ... })`` | Fire-and-forget on a pooled source. Inaudible one-shots are skipped (``null``) unless ``cull: false``. |
| ``source.play(sound, options?)`` | Plays a ``Sound`` through an object's own source; moves with it. Source must be added to a renderer. |
| ``new SpatialAudioRenderer3D(audioDevice, { output: bus })`` / ``source.bus`` / ``source.setBus(bus)`` | Route the renderer, or a single source, to a bus ``Channel``. Sources only cluster with sources on the same bus. |
| ``renderer.listener`` | ``SpatialAudioListener`` (2D) or ``SpatialAudioListener3D`` (3D). |
| ``renderer.output`` / ``renderer.master`` / ``renderer.limiter`` / ``renderer.reverbChannel`` | Output (own master or the ``output`` option), ``master`` (``null`` when the output is a ``Channel``), built-in limiter (``limiter: false`` disables; off by default with an ``output``), reverb bus. |
| ``renderer.setReverbEffect(effect \| null)`` | Default reverb is attached automatically once WASM is ready. |
| ``renderer.clustering`` / ``renderer.options`` | Mutable at runtime. ``options.maxVoices`` caps the voices (64 in 2D, 32 in 3D). |
| ``renderer.getStats()`` | ``{ sources, audible, virtual, voices, clusters, pooledVoices, suspendedLoops }``. |
| ``options.loopVirtualizationDelay`` | Seconds without a voice before looping ``Sound``s of a source are suspended (and later resumed in time). Default ``0.5``, ``Infinity`` disables. |
| ``renderer.getClusters()`` | Debug info per voice. |
| ``(renderer as SpatialAudioRenderer3D).setPanningModel("HRTF" \| "equalpower" \| "stereo")`` | 3D only. |
| ``source.attachAudioClip(clip)`` | Then ``clip.play()``. Do NOT also ``clip.send()`` it to a channel. |
| ``source.attachChannel(channel)`` / ``detachChannel(channel)`` | Positions a channel, e.g. an ``InputChannel`` with a voice (proximity chat). Do NOT also ``send()`` the channel to a master. |
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
3. **A clip plays twice / in the wrong place**: a clip plays into every target it was sent or attached to. Use one ``AudioClip`` per destination (they can share ``AudioSourceData``).
4. **``clip.play()`` returns ``null`` for rapid-fire sounds**: enable ``overrideMaxAudioBufferNodes`` on the pipeline and call ``clip.setMaxAudioBufferSourceNodes(n)``.
5. **Spatial sounds do not move**: ``renderer.update()`` must be called every frame (or ``renderer.start()`` once).
6. **2D sounds are panned the wrong way**: check ``yAxis`` and the rotation convention (``0`` = facing up on screen, clockwise).
7. **Side-scroller: sounds below the player are muffled**: set ``rearLowpassFactor: 1``.
8. **Effects attached to ``renderer.master`` come after the limiter**: create the renderer with ``limiter: false`` and attach your own ``Limiter`` last if you need effects before it.
9. **``renderer.listener`` is not ``AudioContext.listener``**: the renderers never use the Web Audio listener; do not set it.
10. **``BiquadFilterNode`` Q is in dB for lowpass/highpass**: when building native filters yourself, a Butterworth response needs ``Q: -3.0103`` (dB), not ``0.7071``. Allpass, peaking and notch use a linear Q.
11. **Native compressors add makeup gain**: every ``DynamicsCompressorNode`` (and so ``Compressor``) applies an automatic makeup gain from its curve, several dB at typical settings. ``Limiter`` and ``MultibandCompressor`` undo it; ``Compressor`` keeps the native behavior.
12. **Some spatial sources are silent when many sounds play**: by design. Above ``options.maxVoices`` the quietest sources become virtual (``source.isVirtual``). Raise ``maxVoices`` if your target devices can handle it; check ``renderer.getStats()``.
13. **Do not call ``setTargetAtTime`` every frame on many params**: every automation call is a message to the audio thread. The spatial renderer only sends changes above an inaudible threshold; do the same in your own code.
14. **An ``InputChannel`` is silent**: it is not connected to anything by default; ``send()`` it to a master, or ``attachChannel()`` it to a spatial source.
15. **Microphone feedback or echo**: monitoring a microphone through speakers feeds back. ``InputChannel`` disables echo cancellation by default (raw signal for effects); enable it in the options for speech, and always for the microphone you send to other players.
16. **The "side" of a microphone is silent**: a microphone is mono (``L = R``), so ``StereoMono`` ``"side"`` and mid/side splits have nothing to work with until one side is delayed.
17. **A split sounds doubled**: ``StereoMono.split()`` keeps the existing sends of the source channel. ``unsend(master)`` the source if only the branches should be heard.
18. **Remote WebRTC voices are silent in Web Audio**: use ``inputChannel.setMediaStream()``; it adds the muted media element Chromium needs. Do not create a ``MediaStreamAudioSourceNode`` yourself without it.
19. **Hundreds of channels in a game**: do not create a ``Channel`` per game object. Use a few bus channels, one ``SpatialAudioSource`` per object (``bus`` option), ``renderer.playAt()`` for sounds without an owner, and ``Sound`` instead of ``AudioClip`` for sound effects. See [Example 12](./examples/12-game-audio-architecture.md).
20. **``channel.analyserNode`` or ``channel.stereoPannerNode`` is ``null``**: they are created on demand. Call ``channel.enableAnalyser()`` / ``channel.pan(x)`` first.
21. **``renderer.master`` is ``null``**: the renderer was given a ``Channel`` as ``output``. Use ``renderer.output`` or the bus itself.
22. **``playAt()`` returns ``null``**: the one-shot was too far away to hear (pass ``cull: false``), or the sound's ``minInterval``/``maxInstances`` skipped it. Always use ``instance?.``.
23. **A loop played through an ``AudioClip`` on a far away source keeps running**: only ``Sound`` loops are virtualized. Use ``source.play(sound, { loop: true })`` or ``playAt(..., { loop: true })`` for ambience.

---

## 6. Where to look

| Task | Page |
|---|---|
| Install and initialize | [Getting started](../Getting%20started/How%20to%20use%20FluexGL-DSP.md) |
| Spatial audio concepts | [Spatial audio](../Getting%20started/Spatial%20audio.md) |
| Microphones and device switching | [Example 09](./examples/09-input-and-output-devices.md), [InputChannel](./classes/InputChannel.md) |
| Splitting, merging, surround | [Example 10](./examples/10-splitting-and-merging.md), [StereoMono](./effects/StereoMono.md) |
| Voice chat in a game world | [Example 11](./examples/11-proximity-voice-chat.md) |
| Sound effects | [Example 06](./examples/06-one-shot-sounds.md), [Sound](./classes/Sound.md) |
| Organizing a whole game (buses, sources, ambience) | [Example 12](./examples/12-game-audio-architecture.md) |
| Complete examples | [Examples](./examples/README.md) |
| Class reference | [classes/](./classes/README.md) |
| Effect reference | ``effects/<Name>.md`` |
| Option interfaces and types | ``interfaces/<Name>.md`` |
