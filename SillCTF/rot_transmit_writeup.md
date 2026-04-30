# Rot Transmit

Type: Brainrot Apocalypse / Forensics

## Flag

`sillyCTF{r0t_succe33fu11y_transmit3d!}`

## Summary

The short URL did not directly expose a file, but it resolved to a SharePoint / OneDrive folder containing a WAV recording named `rot.wav`.

The audio was not speech. Inspecting the waveform showed two bursts of a clipped digital-looking transmission. The timing was consistent with about `512` symbols per second, which suggested a pager protocol rather than a voice recording.

Once the WAV was resampled to `22050 Hz` raw PCM and fed to `multimon-ng`, it decoded cleanly as a `POCSAG512` alpha pager message and revealed the flag.

## Recon

The challenge provided:

```text
https://shorturl.at/D5B9k
```

That redirected to a OneDrive folder containing `rot.wav`. Basic file identification:

```bash
file rot.wav
```

Output:

```text
rot.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 48000 Hz
```

The recording contains two nearly identical active regions separated by silence, which is typical of a repeated radio/pager burst.

## Decode

`multimon-ng` expects raw PCM at its preferred input rate, so the cleanest path was to resample with `ffmpeg` and pipe the result directly into the POCSAG decoder:

```bash
ffmpeg -loglevel error -i rot.wav -f s16le -ac 1 -ar 22050 - \
  | multimon-ng -a POCSAG512 -f alpha -t raw -
```

Output:

```text
POCSAG512: Address: 1234567  Function: 3  Alpha:   sillyCTF{r0t_succe33fu11y_transmit3d!}   <DLE>
POCSAG512: Address: 1234567  Function: 3  Alpha:   sillyCTF{r0t_succe33fu11y_transmit3d!}   <DLE>
```

The extra `<DLE>` is just a trailing control character after the alpha payload. The flag itself is:

```text
sillyCTF{r0t_succe33fu11y_transmit3d!}
```

## Notes

While reversing the raw bitstream, the transmission also showed the standard POCSAG sync word `0x7CD215D8`, which independently confirmed the protocol identification before using `multimon-ng`.
