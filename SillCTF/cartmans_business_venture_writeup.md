# Cartman's Business Venture Writeup

Challenge type: `forensics / misc`

## Summary

The provided connection was a `shorturl.at` link that redirected to a public OneDrive / SharePoint folder containing a single file: `Cartman_Birthday_Bash.docx`.

Direct unauthenticated requests to the file path returned `403`, but the anonymous share flow issued a `FedAuth` cookie. Reusing that cookie against SharePoint's REST API exposed the shared folder contents and allowed the `.docx` to be downloaded cleanly.

The document itself looked normal on the surface, but the OOXML package contained a `customXml/item1.xml` file with hidden homework answers and a Base64 string in an XML comment. Decoding that string produced the flag.

## Recon

The short URL resolved to this public SharePoint folder share:

```text
https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/mqy5254_psu_edu/IgA3CGtuhRrwSYWakRfRZjQgAUuq3Hx7IsNQT6zZ8jj_tGI?e=mch5Wb
```

The folder path exposed by the redirect was:

```text
/personal/mqy5254_psu_edu/Documents/Cartman's Business Venture
```

## Useful commands

Capture the anonymous share cookie:

```bash
curl -s -v -L -c /tmp/cartman.cookies -b /tmp/cartman.cookies \
  -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36' \
  'https://shorturl.at/pXTZd' -o /dev/null
```

Enumerate the shared folder through SharePoint REST:

```bash
curl -s -b /tmp/cartman.cookies -H 'Accept: application/json;odata=verbose' \
  "https://pennstateoffice365-my.sharepoint.com/personal/mqy5254_psu_edu/_api/web/GetFolderByServerRelativeUrl('/personal/mqy5254_psu_edu/Documents/Cartman''s%20Business%20Venture')/Files"
```

Download the document:

```bash
curl -s -b /tmp/cartman.cookies \
  "https://pennstateoffice365-my.sharepoint.com/personal/mqy5254_psu_edu/_api/web/GetFileByServerRelativeUrl('/personal/mqy5254_psu_edu/Documents/Cartman''s%20Business%20Venture/Cartman_Birthday_Bash.docx')/\$value" \
  -o Cartman_Birthday_Bash.docx
```

Unpack and inspect the OOXML contents:

```bash
unzip -oq Cartman_Birthday_Bash.docx -d /tmp/cartman_docx
sed -n '1,200p' /tmp/cartman_docx/customXml/item1.xml
```

The relevant hidden value in the XML comment was:

```text
c2lsbHlDVEZ7Y2FydG1hbl9ob21ld29ya19zb2x1dGlvbnN9
```

Decode it:

```bash
python3 - <<'PY'
import base64
print(base64.b64decode('c2lsbHlDVEZ7Y2FydG1hbl9ob21ld29ya19zb2x1dGlvbnN9').decode())
PY
```

## Flag

```text
sillyCTF{cartman_homework_solutions}
```
