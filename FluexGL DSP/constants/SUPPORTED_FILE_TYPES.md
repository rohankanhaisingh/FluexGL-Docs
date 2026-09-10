# SUPPORTED_FILE_TYPES

The list of audio MIME types recognized as "known" file types by [``loadAudioSource()``](../helpers/loadAudioSource.md).

```ts
const SUPPORTED_FILE_TYPES: string[];
```

- - -

## About
``SUPPORTED_FILE_TYPES`` is a constant array of MIME type strings. [``loadAudioSource()``](../helpers/loadAudioSource.md) checks a resolved file's MIME type against this list and logs a warning (but still proceeds) when a file's type is not in the list.

## Value

```ts
[
    "audio/aac",
    "audio/mp3",
    "audio/mpeg",
    "audio/ogg",
    "audio/wav",
    "audio/wave",
    "audio/webm"
]
```

## Error and warnings

This constant does not emit any errors or warnings by itself.
