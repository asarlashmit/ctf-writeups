# Pibble

Type: Web / Esoteric

## Flag

`sillyCTF{w@$h_my_b3ll@y}`

## Summary

The challenge page is a tiny static app. The visible dog image only calls `tickle()`, but the page also ships a hidden fullscreen image plus an unused `wash()` function that reveals it. Pulling the hidden image directly and enhancing it exposes the message:

```text
you washed pibble's belly!!!
```

From there, the intended flag is the leetspeak form of that phrase:

```text
sillyCTF{w@$h_my_b3ll@y}
```

I verified the exact spelling through the live challenge submission flow in a browser context.

## Recon

Homepage:

```bash
curl -Ls https://pibble.sillyctf.psuccso.org/
```

Relevant HTML:

```html
<img id="imega" src="Untitled.jpg" alt="A cute pibble" onclick="tickle()">

<div id="image-container" class="hidden">
    <img id="image" src="Untitled(1).png">
</div>
```

JavaScript:

```bash
curl -Ls https://pibble.sillyctf.psuccso.org/script.js
```

Relevant function:

```javascript
function wash() {
    const pibble = document.getElementById('imega');
    const imageContainer = document.getElementById('image-container');
    pibble.style.display = "none";
    imageContainer.classList.remove('hidden');
}
```

So the real payload is the hidden `Untitled(1).png`.

## Extraction

Download the hidden image:

```bash
curl -Ls -O 'https://pibble.sillyctf.psuccso.org/Untitled(1).png'
```

Enhance it so the faint text becomes readable:

```bash
python3 - <<'PY'
from PIL import Image, ImageOps

img = Image.open("Untitled(1).png").convert("L")
ImageOps.autocontrast(img).save("pibble_autocontrast.png")
PY
```

OCRing broad horizontal bands of the enhanced image reveals the line:

```text
you washed pibble's belly!!!
```

That strongly suggests the flag body `w@$h_my_b3ll@y`. Submitting the full candidate:

```text
sillyCTF{w@$h_my_b3ll@y}
```

returns `Correct`.

## Note

Direct raw POSTs to the CTFd attempt API returned `403` during this solve, but same-origin browser submission worked and confirmed the flag.
