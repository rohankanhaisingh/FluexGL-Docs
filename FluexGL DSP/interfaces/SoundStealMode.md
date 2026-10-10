# SoundStealMode

What a [``Sound``](../classes/Sound.md) does when it is played while ``maxInstances`` instances are already playing.

```ts
type SoundStealMode = "oldest" | "quietest" | "none";
```

## Values
- `"oldest"` - Stops the instance that started first (with a 20 ms fade), and plays the new one. Good for rapid-fire sounds such as gunshots.
- `"quietest"` - Stops the quietest instance, but only when the new instance is louder than it; otherwise the new instance is skipped. For spatial instances the loudness includes the distance attenuation, so far away explosions make room for close ones.
- `"none"` - Skips the new instance. Good for sounds that should finish, such as a voice line.

Used by [``SoundOptions.steal``](./SoundOptions.md).
