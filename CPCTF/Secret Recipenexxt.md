# Secret Recipenexxt Writeup

## Challenge Type

Forensics / PCAP

## Summary

The provided file was a small HTTP-only packet capture. Reconstructing the visited website and exporting its assets exposed a JPEG image containing the flag on a handwritten note.

## Steps

1. Download the PCAP and inspect it.

   - `file omelette_recipe.pcap`
   - `capinfos omelette_recipe.pcap`
   - `tshark -r omelette_recipe.pcap -q -z io,phs`

   This showed only 48 packets with a few HTTP objects.

2. List HTTP requests and responses.

   - `tshark -r omelette_recipe.pcap -Y http.request -T fields -e http.request.uri`
   - `tshark -r omelette_recipe.pcap -Y http.response -T fields -e http.content_type`

   The browser fetched:

   - `/`
   - `/styles.css`
   - `/image.jpg`
   - `/favicon.ico`

3. Export the HTTP objects.

   - `tshark -r omelette_recipe.pcap --export-objects http,extracted`

   This produced:

   - `%2f` (HTML)
   - `styles.css`
   - `image.jpg`
   - `favicon.ico`

4. Inspect the reconstructed page.

   The HTML was a normal omelette recipe page. The important section said to use the soy sauce shown in `image.jpg`.

5. Open `image.jpg`.

   The image showed a soy sauce bottle and a handwritten note with the flag.

6. Resolve ambiguous handwritten glyphs using the hint.

   The challenge hint said that `や`, `l`, and `1` are not included in the flag, so the unclear characters should be read as:

   - `I` instead of `l` / `1`
   - `7` instead of `や`

## Flag

`CPCTF{S(I)j-sov(7)}`
