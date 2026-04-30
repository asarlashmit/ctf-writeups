# French Oui Alien

## Challenge Overview
- Event: CyberGeek CTF
- Final Flag: geek{0p3r4t70n8080}

## Executive Summary
The challenge provided a single attachment, [audio.aac](/home/kali/ctf-orc/ctf/french-oui-alien/audio.aac), and the expected flag format `geek{...}`.

Initial triage with `file`, `ffprobe`, and `exiftool` showed that the attachment was just a normal mono AAC stream with no useful metadata. That ruled out the easy path of recovering the flag from tags or appended data and pointed to the audio itself as the carrier.

## Solution Walkthrough
### Recon
Basic checks:

```bash
file audio.aac
ffprobe -hide_banner audio.aac
exiftool audio.aac
```

Results:

- `audio.aac` was an ADTS AAC-LC file
- 44.1 kHz, mono
- about 36 seconds long
- no hidden metadata or obvious embedded strings

The turning point was treating the file like a signal instead of a normal audio recording. A spectrogram showed a highly structured analog pattern rather than speech or music. The frequency range and cadence matched SSTV very closely.

### Identification
The transmission length and tone behavior fit **Robot 36 SSTV**:

- line cadence of roughly `150 ms`
- sync pulses around `1200 Hz`
- separator tones alternating near `1500 Hz` and `2300 Hz`
- total runtime matching a Robot 36 image transmission

That meant the audio was not meant to be listened to directly. It was an image sent over audio.

### Decode
I decoded the AAC stream to PCM, demodulated the SSTV tones, aligned the line timing, and reconstructed the Robot 36 image. The recovered image was a color-bar test pattern with the flag overlaid in the center.

Recovered image artifacts from the decode:

- [decoded_r36_v0.png](/home/kali/ctf-orc/ctf/french-oui-alien/decoded_r36_v0.png)
- [decoded_r36_v1.png](/home/kali/ctf-orc/ctf/french-oui-alien/decoded_r36_v1.png)
- [text_crop.png](/home/kali/ctf-orc/ctf/french-oui-alien/text_crop.png)

The text on the image reads:

```text
geek{0p3r4t70n8080}
```

## Flag
```text
geek{0p3r4t70n8080}
```
