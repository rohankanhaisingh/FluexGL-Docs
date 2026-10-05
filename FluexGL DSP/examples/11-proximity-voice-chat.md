# Example 11: Proximity voice chat

Positions the voices of other players in a 2D game world, so players hear each other from the direction they are standing in, get quieter with distance, and are silent beyond a maximum distance.

## What it shows
- Feeding a remote WebRTC stream into an [``InputChannel``](../classes/InputChannel.md) with ``setMediaStream()``.
- Positioning a channel with [``SpatialAudioSource.attachChannel()``](../classes/SpatialAudioSource.md).
- Effects on a voice before it is spatialized.
- Cleaning up when a player leaves.

## Routing

```
local microphone (getUserMedia, echo cancellation on) -> WebRTC -> other players

remote stream -> InputChannel "Player 2" -> [HighPassFilter] -> SpatialAudioSource -> voice (lowpass, panner)
                                                                                   -> renderer master -> speakers
                                                                                   -> reverb bus
```

## Code

The networking part (signaling, creating the ``RTCPeerConnection``s) is game specific and not part of FluexGL DSP. The code below assumes a ``peer`` per remote player, and a ``players`` map with their positions.

```ts
import { DspPipeline, HighPassFilter, InputChannel, SpatialAudioRenderer2D, SpatialAudioSource } from "@fluex/fluexgl-dsp";

interface RemoteVoice {
    channel: InputChannel;
    source: SpatialAudioSource;
}

const voices = new Map<string, RemoteVoice>();

async function start() {

    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    if (!await pipeline.initializeDpsPipeline()) return;

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    await audioDevice.context.resume();

    // 1. The renderer. Distances are in pixels: full volume within 80px, silent beyond 900px.
    const renderer = new SpatialAudioRenderer2D(audioDevice, {
        refDistance: 80,
        maxDistance: 900,
        reverbMaxSend: 0.2
    });

    // 2. The local microphone is NOT spatialized locally; it is sent to the other players.
    //    Voice processing is important here, otherwise other players hear themselves through your speakers.
    const localStream = await navigator.mediaDevices.getUserMedia({
        audio: { echoCancellation: true, noiseSuppression: true, autoGainControl: true }
    });

    for (const peer of peers.values())
        localStream.getAudioTracks().forEach(track => peer.connection.addTrack(track, localStream));

    // 3. A remote player starts talking.
    for (const [playerId, peer] of peers) {

        peer.connection.addEventListener("track", (event) => {

            const channel = new InputChannel(audioDevice.getContext(), `Player ${playerId}`);
            channel.setMediaStream(event.streams[0]);

            // Optional: effects run before the spatialization.
            channel.addEffect(new HighPassFilter({ cutoff: 120 }));

            // Voices keep their own voice (no clustering), so they stay intelligible.
            const source = renderer.createSource({ label: playerId, clusterable: false });
            source.attachChannel(channel);

            voices.set(playerId, { channel, source });

            // The player left or the connection dropped.
            channel.addEventListener("input-device-lost", () => removeVoice(playerId));
        });
    }

    // 4. Every frame: move the listener (you) and the sources (the other players).
    function frame() {

        renderer.listener.setPosition(me.x, me.y);
        renderer.listener.setRotation(me.rotation); // optional

        for (const [playerId, { source }] of voices) {
            const player = players.get(playerId);

            if (player) source.setPosition(player.x, player.y);
        }

        renderer.update();
        requestAnimationFrame(frame);
    }

    requestAnimationFrame(frame);

    function removeVoice(playerId: string) {

        const voice = voices.get(playerId);

        if (!voice) return;

        renderer.removeSource(voice.source); // fades out, then disposes and detaches the channel
        voice.channel.close();               // does not stop the remote tracks, they belong to WebRTC
        voices.delete(playerId);
    }
}

document.addEventListener("click", start, { once: true });
```

## Testing without a second player

Use your own microphone as a "remote" player, and move it around with the mouse:

```ts
const testVoice = await audioDevice.createInputChannel(null, "Test voice");
const testSource = renderer.createSource({ clusterable: false }).attachChannel(testVoice);

canvas.addEventListener("mousemove", (event) => testSource.setPosition(event.offsetX, event.offsetY));
renderer.listener.setPosition(canvas.width / 2, canvas.height / 2);
renderer.start();
```

Use headphones, otherwise the microphone picks up the speakers.

## Variations

### Radio / walkie-talkie for teammates far away
```ts
import { LowPassFilter } from "@fluex/fluexgl-dsp";

channel.addEffect(new HighPassFilter({ cutoff: 400 }));
channel.addEffect(new LowPassFilter({ cutoff: 3000 }));
source.setAttenuation({ maxDistance: 1e6, rolloffFactor: 0 }); // no distance falloff (use a finite maxDistance, not Infinity)
```

### 3D games
Use [``SpatialAudioRenderer3D``](../classes/SpatialAudioRenderer3D.md) (HRTF, meters) instead. The rest of the code is the same, with ``setPosition(x, y, z)``. See [Example 05](./05-spatial-3d-threejs.md).

## Notes
- Do not ``send()`` a voice channel to a master channel as well: it would also be heard without positioning. ``attachChannel()`` warns about this (``WARNING:FLUEXGL-DSP@0006``).
- ``setMediaStream()`` plays the stream through a muted ``<audio>`` element as well. Chromium based browsers do not deliver remote WebRTC audio to Web Audio without it. The element is not audible.
- ``InputChannel.close()`` never stops the tracks of a stream passed to ``setMediaStream()``: the stream belongs to the peer connection.
- An ended remote stream does not fall back to your own microphone, regardless of ``fallbackToDefault``.
- The voice budget (``maxVoices``) also applies to voices. With ``clusterable: false``, every player within range uses a voice of its own.
