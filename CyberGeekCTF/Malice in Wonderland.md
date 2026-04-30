# Malice in Wonderland

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Polyglot Files
- Artifact(s): artifact.zip, artifact.pdf
- Final Flag: geek{p0lygl0t_ch4m3l3on_m4st3r_2026}

## Executive Summary
The attachment is a polyglot file. `artifact.zip` extracts to a single PDF, but the PDF has two ZIP archives appended after the logical end of the document.

The visible PDF content is a distraction: nine identical `Cybergeek CTF` banners. The actual solve path comes from inspecting the file tail.

## Solution Walkthrough
### Recon
Download and extract the attachment:

```bash
curl -L -o artifact.zip https://files.ctf7.com/media/challenge_attachments/artifact.zip
unzip -o artifact.zip
```

Basic inspection shows only a one-page PDF:

```bash
exiftool artifact.pdf
mutool draw -F png -o page.png artifact.pdf 1
```

Rendering the page shows repeated banner images and nothing flag-like.

### Finding the Payload
Running `binwalk` against the PDF reveals appended ZIP structures:

```bash
binwalk artifact.pdf
```

Relevant output:

```text
106840  Zip archive data, encrypted, name: hint.txt
107132  Zip archive data, name: __main__.py
```

`unzip` confirms the file is a valid PDF/ZIP polyglot:

```bash
unzip -l artifact.pdf
```

That lists `__main__.py` even though the file extension is `.pdf`.

### Extracting the Logic
Carve out the second ZIP and inspect the Python entrypoint:

```bash
python3 - <<'PY'
from pathlib import Path
b = Path("artifact.pdf").read_bytes()
Path("zipapp.zip").write_bytes(b[107132:])
PY

unzip -p zipapp.zip __main__.py
```

`__main__.py` is a base64-wrapped Python script. Decoded, it becomes:

```python
import socket
def start():
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        s.bind(('0.0.0.0', 9999))
        s.listen()
        c, a = s.accept()
        if c.recv(1024).decode().strip() == "OPEN_SESAME":
            c.sendall(b"geek{p0lygl0t_ch4m3l3on_m4st3r_2026}\n")
    except: pass
start()
```

So the “heart that beats only when it's told to” hint refers to this listener: it stays idle until it receives the correct trigger string.

### Getting the Flag
Run the zipapp:

```bash
python3 zipapp.zip
```

In another terminal, connect and send the trigger:

```bash
python3 - <<'PY'
import socket
s = socket.create_connection(("127.0.0.1", 9999))
s.sendall(b"OPEN_SESAME\n")
print(s.recv(4096).decode(), end="")
PY
```

Output:

```text
geek{p0lygl0t_ch4m3l3on_m4st3r_2026}
```

## Flag
```text
geek{p0lygl0t_ch4m3l3on_m4st3r_2026}
```

## References and Notes
### Notes
- The encrypted `hint.txt` ZIP is not required to recover the flag.
- The intended trick is recognizing the artifact as a polyglot PDF with appended executable content.
