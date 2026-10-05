# Example 09: Input and output devices

Records a microphone through an effect chain, and lets the user switch the microphone and the output device at runtime, without reloading the page.

## What it shows
- Opening a microphone on an [``InputChannel``](../classes/InputChannel.md) with [``audioDevice.createInputChannel()``](../classes/AudioDevice.md).
- Listing devices with [``listAudioInputDevices``](../helpers/listAudioInputDevices.md) and [``listAudioOutputDevices``](../helpers/listAudioOutputDevices.md).
- Switching the microphone with ``inputChannel.setInputDevice()`` and the output with ``audioDevice.setOutputDevice()``.
- Keeping the menus up to date with [``watchAudioDevices``](../helpers/watchAudioDevices.md) and the device events.

## Routing

```
Microphone -> InputChannel "Microphone" -> [Compressor] -> Master -> selected output device
```

## Code

```ts
import { AudioDevice, Compressor, DspPipeline, listAudioInputDevices, listAudioOutputDevices, watchAudioDevices } from "@fluex/fluexgl-dsp";

const inputSelect = document.querySelector<HTMLSelectElement>("#input")!;
const outputSelect = document.querySelector<HTMLSelectElement>("#output")!;

function fill(select: HTMLSelectElement, devices: MediaDeviceInfo[], selectedId: string) {
    select.replaceChildren(...devices.map(device => new Option(device.label || device.deviceId, device.deviceId)));
    select.value = selectedId;
}

async function start() {

    const pipeline = new DspPipeline({
        pathToWasm: "/bin/fluexgl-dsp-wasm_bg.wasm",
        pathToWorklet: "/bin/fluexgl-dsp-processor.worklet"
    });

    if (!await pipeline.initializeDpsPipeline()) return;

    const audioDevice = await pipeline.resolveDefaultAudioOutputDevice();

    if (!audioDevice) return;

    await audioDevice.context.resume();

    // 1. Open the default microphone on its own channel. It is not connected to anything yet.
    const microphone = await audioDevice.createInputChannel(null, "Microphone");

    microphone.addEffect(new Compressor({ threshold: -24, ratio: 3 }));
    microphone.send(audioDevice.getMasterChannel()); // Use headphones to avoid feedback.

    // 2. Device menus.
    fill(inputSelect, await listAudioInputDevices(), microphone.deviceInfo?.deviceId ?? "");
    fill(outputSelect, await listAudioOutputDevices(), audioDevice.outputDeviceId || "default");

    outputSelect.disabled = !AudioDevice.supportsOutputDeviceSelection;

    // 3. Switching. The channel, its effects and its sends stay intact.
    inputSelect.addEventListener("change", () => microphone.setInputDevice(inputSelect.value));
    outputSelect.addEventListener("change", () => audioDevice.setOutputDevice(outputSelect.value));

    // 4. Keep the menus up to date when devices are plugged in or out.
    watchAudioDevices(({ inputs, outputs }) => {
        fill(inputSelect, inputs, microphone.deviceInfo?.deviceId ?? "");
        fill(outputSelect, outputs, audioDevice.outputDeviceId || "default");
    });

    // When a selected device is unplugged, the library falls back to the default device
    // and fires these events, so the menus can follow.
    microphone.addEventListener("input-device-changed", ({ current }) => {
        inputSelect.value = current?.deviceId ?? "";
    });

    audioDevice.addEventListener("output-device-changed", ({ current }) => {
        outputSelect.value = current?.deviceId ?? "default";
    });
}

document.addEventListener("click", start, { once: true });
```

## Muting and closing

```ts
microphone.setMuted(true);  // keeps the device open, sends silence
microphone.setMuted(false);

microphone.close();         // releases the device (the browser's recording indicator turns off)
await microphone.open();    // opens the default device again

audioDevice.removeInputChannel(microphone); // closes it and removes all its sends
```

## Notes
- Echo cancellation, noise suppression and automatic gain control are **disabled** by default, so effects get the raw signal. Enable them with the options of [``InputChannelOptions``](../interfaces/InputChannelOptions.md) when recording speech.
- Switching the output device requires ``AudioContext.setSinkId()``, which is available in Chromium based browsers. In other browsers ``setOutputDevice()`` logs ``WARNING:FLUEXGL-DSP@0002`` and returns ``false``.
- Device labels are empty until the user granted microphone permission. ``initializeDpsPipeline()`` asks for it.
- A microphone is mono. Every channel upmixes it to both speakers, so it is heard in the center.
