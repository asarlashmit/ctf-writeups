# Musical Genius

Category: Forensics

## Summary

The provided short URL led to a public OneDrive folder containing a single WAV file, `Groovy_Beat.wav`.

The audio itself was not archive-stuffed and did not contain an obvious LSB payload. The intended trick was the spectrogram: rendering the WAV as a frequency-vs-time image revealed the flag text directly.

## Recon

The challenge API showed:

- Name: `Musical Genius`
- Type: `Forensics`
- Connection: `https://shorturl.at/mIAZ6`

The short URL was Cloudflare-protected for plain `curl`, so I resolved it with headless Chromium. That redirected to a public OneDrive folder named `groovy`, which exposed `Groovy_Beat.wav`.

## Artifact Retrieval

I used Chromium/Puppeteer to establish the anonymous SharePoint guest session, then downloaded the file with the resulting `FedAuth` cookie.

Relevant facts about the file:

- RIFF/WAV PCM
- Stereo
- 22050 Hz
- Duration: about 5.0 seconds

## Analysis

Initial checks showed nothing special in the container:

- `binwalk` found only normal WAV structure
- `strings` showed no flag
- Simple LSB inspection did not reveal readable data

The next step was to render a spectrogram:

```bash
ffmpeg -y -hide_banner -i Groovy_Beat.wav \
  -lavfi showspectrumpic=s=1600x800:legend=disabled \
  Groovy_Beat_spectrogram.png
```

The spectrogram clearly displayed:

```text
sillyCTF{LA_horrible_beat}
```

## Flag

```text
sillyCTF{LA_horrible_beat}
```
