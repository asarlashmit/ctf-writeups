# Real Time Rizz

Type: Brainrot Apocalypse / Web

## Summary

The RTSP stream was not plain text or QR code data. Capturing a short sample and stepping through frames showed a sequence of **I Ching hexagrams**. The optional hint confirmed the right direction:

> I heard King Wen is on Twich now

`King Wen` refers to the standard **King Wen ordering** of the 64 hexagrams.

The solve was:

1. Capture the RTSP stream with `ffmpeg`
2. Extract frames
3. Read each hexagram as six yin/yang lines
4. Convert each hexagram to its **King Wen number** (`1..64`)
5. Treat those 64 symbols as a Base64 alphabet (`1 -> A`, `2 -> B`, ..., `64 -> /`)
6. Decode the repeating Base64 payload

That produced the flag:

```text
sillyCTF{RizzTSP042}
```

## Recon

The stream is:

```text
rtsp://rizz-stream.psuccso.org:30857/stream
```

Probe it:

```bash
ffprobe -rtsp_transport tcp rtsp://rizz-stream.psuccso.org:30857/stream
```

It is a 1080p H.264 video stream with AAC audio.

Capture a local sample:

```bash
ffmpeg -rtsp_transport tcp -i rtsp://rizz-stream.psuccso.org:30857/stream -t 60 -c copy rizz_long.mkv
```

Extract frames:

```bash
ffmpeg -i rizz_long.mkv -vf fps=4 rizz_4fps/f_%03d.jpg
```

## Visual Pattern

The important frames show six horizontal lines, some solid and some broken. Those are hexagrams from the *I Ching*.

For example, the first few decoded hexagrams from the sample were:

```text
26, 60, 21, 39, 38, 59, 31, 38, 18, 20, 21, 4, 1, 53, ...
```

These are **King Wen sequence numbers**, not binary-order hexagram numbers.

## Decode

Map each King Wen number into a Base64 character by subtracting 1 and indexing:

```text
1  -> A
2  -> B
...
26 -> Z
27 -> a
...
64 -> /
```

The captured sequence produced a repeating Base64-looking string:

```text
Z7Uml6elRTUDA0Mn0c2lsbHlDVEZ7Uml6elRTUDA0Mn0c2lsbHlDVEZ7Uml
```

Inside that repeated stream is the real payload:

```text
c2lsbHlDVEZ7Uml6elRTUDA0Mn0=
```

Decode it:

```bash
printf 'c2lsbHlDVEZ7Uml6elRTUDA0Mn0=' | base64 -d
```

Output:

```text
sillyCTF{RizzTSP042}
```

## Flag

```text
sillyCTF{RizzTSP042}
```
