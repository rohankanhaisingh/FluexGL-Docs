# LoadAudioSource

An essential helper function within FluexGL DSP for loading audio files from a specified path.

```ts
async function LoadAudioSource(path: string, options: Partial<LoadAudioSourceOptions>): Promise<AudioSourceData | null>;
```

- - - 

## About
The ``LoadAudioSource()`` function loads an audio file from a given path and decodes it into an AudioBuffer using the Web Audio API.
It verifies the file type using MIME type detection and performs error handling when the file type or path cannot be resolved.

Note: This function is asynchronous, which means you must call it within an asynchronous scope..

## Parameters
- ``path``: ``string`` - The file path or URL to the audio source.
- ``options``: [``Partial<LoadAudioSourceOptions>``](https://github.com/rohankanhaisingh/FluexGL-Docs/tree/master/api/fluexgl-dsp/interfaces/LoadAudioSourceOptions.md) - Optional configuration that controls how the function handles unknown or unsupported file types.

## Returns (promised)

- [``AudioSourceData``](https://github.com/rohankanhaisingh/FluexGL-Docs/tree/master/api/fluexgl-dsp/interfaces/AudioSourceData.md) - An object containing the decoded audio data and metadata.
- ``null`` - Null if loading failed.

## Error and warnings

### ``ERROR:FLUEXGL-DSP@0002``
The file type could not be determined and foreign file types are not allowed. ``ErrorCodes.NO_FILE_TYPE_FOUND``

### ``ERROR:FLUEXGL-DSP@0003``
The specified file could not be found or loaded. ``ErrorCodes.PATH_TO_FILE_NOT_FOUND``

### Unknown warning
If the file type is not recognized but ``allowForeignFileTypes`` is ``true``, a warning will be printed:
``"The file type of the specified file is unknown and possibly unsupported, but will be used anyways."``.