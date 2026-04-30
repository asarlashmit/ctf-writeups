# ISpyLie Writeup

## Challenge

- Name: `ISpyLie`
- Type: audio / misc

## Recon

The provided file was a plain PCM WAV:

```text
RIFF (little-endian) data, WAVE audio, Microsoft PCM, 8 bit, mono 8000 Hz
```

There was no useful metadata, no embedded files, and no printable payload in the container.

## Signal Analysis

Plotting the spectrogram showed a steady carrier around `800 Hz`, which suggested the file was not speech and was likely an encoded tone stream.

Looking at the raw samples showed that the waveform alternated between:

- exact silence (`0x80` / decimal `128` in unsigned 8-bit PCM)
- a repeated 800 Hz waveform

Measuring the run lengths of silence and signal gave two dominant durations:

- short: about `469-491` samples, roughly `60 ms`
- long: about `1429-1451` samples, roughly `180 ms`

That is a classic Morse timing pattern:

- short signal = dot
- long signal = dash
- short gap = intra-character separator
- long gap = character separator

## Decode

Decoding the on/off keyed Morse stream produced:

```text
VMLZAHDHQ1RGE20WCJUZX2YZM2W1X2MWMGX9
```

That string looks random at first, but it has an important property:

- it is valid uppercase-only base64 text
- base64 of `VishwaCTF{...}` normally starts with `VmlzaHdhQ1RG...`
- uppercasing that becomes `VMLZAHDHQ1RG...`, which matches the Morse output exactly

So the Morse layer did not directly reveal the flag. It revealed an uppercased version of the flag's base64 encoding. The lost lowercase/uppercase information has to be restored per 4-character base64 block.

Reconstructing the original base64 casing gives:

```text
VmlzaHdhQ1RGe20wcjUzX2YzM2w1X2MwMGx9
```

Decoding that base64 string yields the real flag:

```text
VishwaCTF{m0r53_f33l5_c00l}
```

## Reproduction

Run:

```bash
python3 solve.py
```

It prints both the decoded token and the wrapped flag.
