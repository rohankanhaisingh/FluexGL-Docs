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

await analyser.InitializeOnAttachment(audioContext);

const waveform = analyser.GetWaveformFloatData();
const spectrum = analyser.GetFrequencyByteData();
```

- - -

## Constructor

```ts
new Analyser(options?: Partial<AnalyserOptions>): Analyser;
```

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

### ``InitializeOnAttachment(context: AudioContext): Promise<void>``

### ``SetOptions(options: Partial<AnalyserOptions>): void``

### ``GetWaveformFloatData(): Float32Array | null``

### ``GetWaveformByteData(): Uint8Array | null``

### ``GetFrequencyFloatData(): Float32Array | null``

### ``GetFrequencyByteData(): Uint8Array | null``
