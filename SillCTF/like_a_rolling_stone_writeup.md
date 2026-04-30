# Like A Rolling Stone

Category: Listen Up

## Summary

The short URL redirected to a public OneDrive folder named `Bob Dylan` containing a single file, `LikeARollingStone.wav`.

The WAV itself was plain PCM audio and did not have an obvious appended archive or readable strings payload. The useful clue was the spectrogram: the end of the track contained a highly structured block rather than normal music.

That trailing block was a hidden signal. After isolating it and speeding it up by roughly `4.7x`, the spectrogram of the resulting short clip displayed the flag text directly.

## Recon

- Challenge name: `Like A Rolling Stone`
- Connection: `https://shorturl.at/5X8EG`
- Short URL hint: Bob Dylan / “more funky”

The shortener is Cloudflare-protected for plain `curl`, so I resolved it with headless Chromium. That chain landed on a public SharePoint/OneDrive folder:

- Folder: `Bob Dylan`
- File: `LikeARollingStone.wav`

## Artifact Retrieval

After the browser established the anonymous SharePoint guest session, I pulled the WAV through the OneDrive drive API.

Basic file facts:

- RIFF/WAV PCM
- Mono
- 44100 Hz
- Duration: about `76.6s`

## Analysis

Initial container checks were uninteresting:

- `file` / `ffprobe` only showed a normal PCM WAV
- `strings` did not reveal a flag
- `binwalk` did not show a useful appended payload

Rendering a spectrogram of the original file showed a strange dense block near the end, which suggested a hidden time-compressed audio payload rather than normal song content.

The solve path was:

1. Isolate the trailing structured section from the original WAV.
2. Speed that section up by about `4.6875x` to turn it into a short “beat”.
3. Render a spectrogram of the derived clip.

Example commands:

```bash
ffmpeg -y -hide_banner -loglevel error \
  -ss 52.9 -i LikeARollingStone.wav -t 23.7 \
  -ac 1 -ar 48000 extracted_tail.wav

ffmpeg -y -hide_banner -loglevel error \
  -i extracted_tail.wav \
  -filter:a atempo=2.0,atempo=2.0,atempo=1.171875 \
  tail_rate46875.wav
```

The derived clip in the workspace as `Groovy_Beat.wav` rendered cleanly with:

```bash
ffmpeg -y -hide_banner -loglevel error \
  -i Groovy_Beat.wav \
  -lavfi "showspectrumpic=s=1600x800:legend=disabled" \
  Groovy_Beat_spectrogram.png
```

That spectrogram displayed:

```text
sillyCTF{LA_horrible_beat}
```

## Flag

```text
sillyCTF{LA_horrible_beat}
```
