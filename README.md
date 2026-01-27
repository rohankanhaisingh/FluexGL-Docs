# FluexGL

## Overview
FluexGL is a client-side toolkit for building advanced, WebGPU-powered graphics and for manipulating high-performance, Rust-written, engine-driven audio without noticeable latency.

FluexGL and FluexGL DSP are named after the company Fluex. It’s just a dope name, lol.

If you have experience with libraries such as Three.js, Tone.js, or native APIs like Web Audio and Web Graphics, you’ll feel right at home here.

## Features
- 📘 TypeScript-written libraries  
- 🖼️ Advanced graphics processing using WebGPU  
- ❗ Detailed error handling  
- 🎶 Advanced audio processing using a Rust-written engine  
- ⚡ Medium-sized library with optimized performance across platforms  

## Introduction
The primary goal of FluexGL is to bring advanced graphics to the web in a way that is easy to understand, maintain, and scale. Many modern libraries lack consistency, often forcing developers to achieve the same result through large batches of low-level calls. FluexGL focuses on object-based abstractions that are highly customizable and developer-friendly.

FluexGL DSP is an additional library within the FluexGL project, focusing on high-performance audio processing on the web using a Rust- and TypeScript-written audio engine. FluexGL DSP integrates seamlessly with FluexGL, enabling the creation of rich, interactive scenes such as games and audiovisual experiences.

## Why is FluexGL not available yet? (Q1 2026)
As of today (27-01-2026), the core functionality of FluexGL is not yet publicly available, as it is still under heavy development. FluexGL DSP, however, *is* already available. FluexGL itself is expected to be released publicly around the third quarter of 2026.

## Why is the DSP worklet so large in size?
The processed files (WebAssembly modules and audio worklets) are relatively large because they include code from multiple open-source libraries. Instead of relying on shared libraries between the main thread and the audio thread, each worklet contains the full required implementation. This design choice prioritizes performance and minimizes runtime overhead.

## How is FluexGL different from three.js?
FluexGL closely follows the architectural concepts of three.js, or at least takes heavy inspiration from it. The main difference lies in the underlying rendering technology: three.js primarily relies on WebGLRenderingContext, whereas FluexGL is built on top of the newer WebGPU API.

## How is FluexGL DSP different from other DSP libraries?
FluexGL DSP uses a unique approach to audio processing. Instead of processing audio on the main thread, it leverages the AudioContext and custom processor nodes to run audio processing on a separate thread using WebAssembly. This prevents bottlenecks and keeps the main thread responsive.
