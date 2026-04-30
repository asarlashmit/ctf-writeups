# Six to the Seven

## Challenge
- Name: `Six to the Seven`
- Type: unknown

## Flag
- `sillyCTF{haha_six_seven}`

## Summary
The short URL expanded to an anonymously shared SharePoint folder. The folder contained a PowerPoint file, `sixtotheseven.pptx`. The deck used three reused image frames across 215 slides. The intended payload was not in the image pixels themselves; it was in the slide sequence.

## Solve Steps
1. Expanded the short URL and followed the redirect chain.
   - The useful target was a SharePoint folder share under `pennstateoffice365-my.sharepoint.com`.
2. Reused the anonymous `FedAuth` cookie from the share redirect.
   - Loading the shared `onedrive.aspx` page as a guest exposed `_spPageContextInfo`.
3. Extracted the guest OneDrive API details from `_spPageContextInfo.driveInfo`.
   - `.driveUrl`
   - `.driveAccessTokenV21`
4. Queried the SharePoint drive API on `/_api/v2.1/...`.
   - Enumerated the shared folder `627`
   - Downloaded `sixtotheseven.pptx`
5. Unzipped the PowerPoint and inspected slide relationships.
   - `image2.png` corresponded to `6`
   - `image3.png` corresponded to `7`
   - `image4.jpg` was the separator
6. The important detail was slide order.
   - Sorting `slide*.xml` filenames produced garbage
   - The real order came from:
     - `ppt/presentation.xml`
     - `ppt/_rels/presentation.xml.rels`
7. Rebuilt the stream in actual presentation order.
   - This yielded 24 groups, each exactly 8 symbols long
   - Mapping `6 -> 0` and `7 -> 1` decoded directly to ASCII

## Recovered Byte Stream
`67776677/67767667/67767766/67767766/67777667/67666677/67676766/67666776/67777677/67767666/67766667/67767666/67766667/67677777/67776677/67767667/67777666/67677777/67776677/67766767/67776776/67766767/67767776/67777767`

## Decode
Using `6 -> 0` and `7 -> 1`:

`01110011 01101001 01101100 01101100 01111001 01000011 01010100 01000110 01111011 01101000 01100001 01101000 01100001 01011111 01110011 01101001 01111000 01011111 01110011 01100101 01110110 01100101 01101110 01111101`

ASCII:

`sillyCTF{haha_six_seven}`

## Minimal Python Decoder
```python
import os
import xml.etree.ElementTree as ET

NS = {
    "p": "http://schemas.openxmlformats.org/presentationml/2006/main",
    "r": "http://schemas.openxmlformats.org/officeDocument/2006/relationships",
}

pres = ET.parse("ppt/presentation.xml").getroot()
rels = ET.parse("ppt/_rels/presentation.xml.rels").getroot()
rmap = {rel.attrib["Id"]: rel.attrib["Target"] for rel in rels}

stream = []
for sld in pres.find("p:sldIdLst", NS):
    rid = sld.attrib["{http://schemas.openxmlformats.org/officeDocument/2006/relationships}id"]
    target = rmap[rid]
    rel_path = "ppt/slides/_rels/" + os.path.basename(target) + ".rels"
    txt = open(rel_path, "r", encoding="utf-8").read()
    if "image2.png" in txt:
        stream.append("6")
    elif "image3.png" in txt:
        stream.append("7")
    elif "image4.jpg" in txt:
        stream.append("/")

groups = "".join(stream).split("/")
bits = ["".join("0" if c == "6" else "1" for c in group) for group in groups]
flag = "".join(chr(int(b, 2)) for b in bits)
print(flag)
```
