# L0v3 PDF Writeup

## Challenge

- Name: `L0v3 PDF`
- Type: `forensics / PDF structure`

## Recon

The downloaded file is a one-page PDF:

```bash
file il0v3pdfs.pdf
```

Relevant result:

```text
il0v3pdfs.pdf: PDF document, version 1.7, 1 page(s)
```

Extracting visible text shows a dummy flag:

```bash
mutool draw -F txt il0v3pdfs.pdf
```

Output:

```text
CPCTF{dummy!}
```

## Analysis

To inspect the raw page content stream:

```bash
mutool show il0v3pdfs.pdf 3
```

This reveals:

```text
3 0 obj
<<
  /Length 119
>>
stream
BT
/F42 24.7871 Tf 148.712 699.875 Td [(CPCTF{dummy!})]TJ
ET
1 0 0 1 327.811 699.875 cm
 % CPCTF{Lets_P1ey_W1th_PdFs}
endstream
endobj
```

The important detail is the `%` character. In a PDF content stream, `%` starts a comment, so that line is not rendered on the page. The author placed the real flag inside a PDF comment, while the displayed text contains a decoy flag.

## Flag

```text
CPCTF{Lets_P1ey_W1th_PdFs}
```
