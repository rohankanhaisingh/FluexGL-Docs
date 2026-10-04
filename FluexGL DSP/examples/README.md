# FluexGL DSP examples

Complete, copy-pasteable examples. Every example is self-contained: it shows the imports, the setup and the per-frame code. They build on each other, so if you are new to FluexGL DSP, start at the top.

New to the library, or an AI model working with it? Read the [AI guide](../AI-GUIDE.md) first. It explains the mental model and the pitfalls in one page.

| # | Example | Shows |
|---|---|---|
| 01 | [Basic playback](./01-basic-playback.md) | Pipeline, audio device, channels, playing a clip |
| 02 | [Effect chains](./02-effect-chains.md) | Worklet and native effects, compressor, removing effects |
| 03 | [Spatial 2D: top-down game](./03-spatial-2d-top-down.md) | 2D renderer, moving sources, rotating listener |
| 04 | [Spatial 2D: side-scroller](./04-spatial-2d-side-scroller.md) | 2D renderer without rotation, ``yAxis``, ``rearLowpassFactor`` |
| 05 | [Spatial 3D with three.js](./05-spatial-3d-threejs.md) | 3D renderer, HRTF, following a camera |
| 06 | [One-shot sounds](./06-one-shot-sounds.md) | Gunshots, footsteps, temporary sources |
| 07 | [Debugging clusters](./07-debugging-clusters.md) | ``source.state``, ``getClusters()``, tuning clustering |
| 08 | [Mixing and loudness](./08-mixing-and-loudness.md) | Music + world + UI, limiter, compressor, multiple renderers |

All examples assume the WebAssembly and worklet files are served under ``/bin/``. See [Getting started](../../Getting%20started/How%20to%20use%20FluexGL-DSP.md) for how to get them.
