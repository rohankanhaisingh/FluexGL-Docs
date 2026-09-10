# Class ``Analyser``

An audio analyser effector that can be attached in the DSP graph to read **waveform** (time-domain) and **frequency** (spectrum) data using the Web Audio API's ``AnalyserNode``.

It exposes pre-allocated typed-array buffers (Float32 and Uint8 variants) and provides helper methods to fill and return those buffers.

## Example

```ts
import { Analyser } from "@fluex/fluexgl-dsp";

const analyser = new Analyser({
    fftSize: 2048,
    smoothingTimeConstant: 0.85,
    minDecibels: -90,
    maxDecibels: -10
});

channel.addEffect(analyser);

const waveform = analyser.getWaveformFloatData();
const spectrum = analyser.getFrequencyByteData();
```

- - -

## Constructor

```ts
new Analyser(options?: Partial<AnalyserOptions>): Analyser;
```

``AnalyserOptions`` here is the standard Web Audio API type (``fftSize``, ``smoothingTimeConstant``, ``minDecibels``, ``maxDecibels``), not a FluexGL-specific interface. Defaults to ``{ fftSize: 32, smoothingTimeConstant: 0.8, minDecibels: -90, maxDecibels: -10 }``.

### Arguments
- ``options?``: ``Partial<AnalyserOptions>`` - Optional analyser configuration. Only ``fftSize`` values supported by the Web Audio API (``32``–``32768``, power of two) are applied.

- - -

## Properties

- ``label: string | null``
- ``name: string``
- ``analyserNode: AnalyserNode | null``
- ``waveformFloat32ArrayBuffer: Float32Array``
- ``waveformUint8ArrayBuffer: Uint8Array``
- ``frequencyFloat32ArrayBuffer: Float32Array``
- ``frequencyUint8ArrayBuffer: Uint8Array``

- - -

## Methods

### ``initializeOnAttachment(context: AudioContext): Promise<void>``
Creates the underlying ``AnalyserNode`` from the configured options and (re)allocates the typed-array buffers to match its ``fftSize``.

### ``setOptions(options: Partial<AnalyserOptions>): void``
Applies new analyser options. Rejects unsupported ``fftSize`` values. If ``fftSize`` changes and the node already exists, the typed-array buffers are reallocated.

### ``getWaveformFloatData(): Float32Array | null``
Fills and returns ``waveformFloat32ArrayBuffer`` with time-domain data (``getFloatTimeDomainData``). Returns ``null`` if not yet attached.

### ``getWaveformByteData(): Uint8Array | null``
Fills and returns ``waveformUint8ArrayBuffer`` with time-domain data (``getByteTimeDomainData``). Returns ``null`` if not yet attached.

### ``getFrequencyFloatData(): Float32Array | null``
Fills and returns ``frequencyFloat32ArrayBuffer`` with frequency-domain data (``getFloatFrequencyData``). Returns ``null`` if not yet attached.

### ``getFrequencyByteData(): Uint8Array | null``
Fills and returns ``frequencyUint8ArrayBuffer`` with frequency-domain data (``getByteFrequencyData``). Returns ``null`` if not yet attached.
