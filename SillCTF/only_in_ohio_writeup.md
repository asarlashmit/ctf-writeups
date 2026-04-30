# Only in Ohio Writeup

## Challenge
- Name: Only in Ohio
- Type: Unknown

## Flag
- `sillyCTF{toilet_in_ohio}`

## Recon
The provided URL was `https://shorturl.at/OGdOc`.

Direct `curl` requests only returned a Cloudflare challenge page, so the first useful step was using a browser-impersonating HTTP client instead of a normal request client.

Using `curl_cffi` with Chrome impersonation resolved the short URL successfully and revealed the real target:

`https://pennstateoffice365-my.sharepoint.com/personal/mqy5254_psu_edu/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Fmqy5254%5Fpsu%5Fedu%2FDocuments%2Fskibidy&ga=1`

That showed the challenge content lived in a public OneDrive folder named `skibidy`.

## Artifact Extraction
Inside that folder was a file named `SKIBIDITY.txt`.

With the anonymous guest cookies set by the redirected request, the file could be downloaded directly from:

`/personal/mqy5254_psu_edu/Documents/skibidy/SKIBIDITY.txt`

The file contents were mostly ASCII art, but the bottom two lines contained valid Brainfuck instructions embedded inside the art.

## Solving
The trick was to ignore the ASCII-art noise and keep only the Brainfuck characters from the final two lines:

```brainfuck
+++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++++++++++++.----------.+++..+++++++++++++.<---.+++++++++++++++++.--------------.>++.-------.-----.------.+++.-------.+++++++++++++++.---------------------.++++++++++.+++++.---------------.++++++++++++++++.-------.+.++++++.++++++++++++++.+
```

Running that Brainfuck program prints:

```text
sillyCTF{toilet_in_ohio}
```

## Notes
An earlier attempt that stripped Brainfuck characters from the entire file produced garbage output because the ASCII art also contained many `+` and `-` characters. Restricting the extraction to the actual program region fixed the decode immediately.
