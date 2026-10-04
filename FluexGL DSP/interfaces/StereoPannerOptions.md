# StereoPannerOptions

StereoPanner configuration.

```ts
interface StereoPannerOptions {
    pan: number;
    width: number;
}
```

## About
Used by [``StereoPanner``](../effects/StereoPanner.md).

## Properties
- `pan`: `number` - Position, between ``-1`` (left) and ``1`` (right). Default ``0``.
- `width`: `number` - Stereo width, between ``0`` (mono) and ``2`` (extra wide). ``1`` leaves the image unchanged. Default ``1``.
