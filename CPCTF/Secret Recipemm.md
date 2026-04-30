# Secret Recipemm Writeup

## Challenge Type

Forensics / Network (`pcap`)

## Recon

The challenge provides a packet capture. A quick summary with `capinfos` and
`tshark` shows only a small HTTP session:

- `GET /`
- `GET /styles.css`
- `GET /image.jpg`
- `GET /favicon.ico` (404)

So the useful content is just the downloaded web page and its image.

## Extracting the Web Objects

I exported the HTTP objects from the capture:

```bash
tshark -r omelette_recipe.pcap --export-objects http,extracted
```

This yielded:

- `%2f` (the HTML page)
- `styles.css`
- `image.jpg`
- `favicon.ico`

The HTML page is a Japanese omelette recipe. The interesting part is the
"secret tip" section:

```html
<section class="card note">
  <h2>秘密のコツ</h2>
  <p>醤油はこの醤油を使う</p>
  <center><img src="./image.jpg"></center>
</section>
```

So the real clue is inside `image.jpg`.

## Image Analysis

Viewing the extracted JPEG shows a soy sauce bottle and, at the bottom, a
handwritten note that starts with `CPCTF{`.

The raw handwriting is deliberately misleading. The challenge hint says that if
the flag appears to contain characters such as `(`, `)`, `l`, `1`, or `-`, then
it should be reread carefully, and `-` is not included in the flag.

After cropping / enlarging the note and reading it together with the context of
the image (the bottle label is `かけしょうゆ`, "kake shoyu"), the handwritten
flag resolves to:

```text
CPCTF{kakeshoyu}
```

## Flag

```text
CPCTF{kakeshoyu}
```
