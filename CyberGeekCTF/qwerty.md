# qwerty

## Challenge Overview
- Event: CyberGeek CTF
- Category: Web / JavaScript
- Final Flag: geek{k0h_nah_m3}

## Executive Summary
The Vercel app ships its secret in an obfuscated client-side script. Deobfuscating that JavaScript exposes both the trigger sequence and the final flag.

## Solution Walkthrough
### Recon
The home page is a small static Vercel app:

```bash
curl -i -sS https://make-me-dance-five.vercel.app/
```

The HTML loads one JavaScript file:

```html
<script src="scrip.obf.js"></script>
```

I fetched that script:

```bash
curl -sS https://make-me-dance-five.vercel.app/scrip.obf.js
```

### Analysis
The script is obfuscated, but the important behavior is still visible after inspection. It defines a keyboard sequence and a flag value:

```js
const konamiCode = [
  "ArrowUp",
  "ArrowUp",
  "ArrowDown",
  "ArrowDown",
  "ArrowLeft",
  "ArrowRight",
  "ArrowLeft",
  "ArrowRight",
  "b",
  "a"
];

flagValue = "geek{k0h_nah_m3}";
```

The event handler listens for `keydown` events. When the full sequence is entered, `triggerDance()` runs and writes the flag into the hidden `secret` element:

```js
secretEl.textContent = flagValue;
secretEl.classList.remove("hidden");
```

So the page is a browser-based Konami-code challenge. The intended interaction is:

```text
Up, Up, Down, Down, Left, Right, Left, Right, B, A
```

### Exploit
The fastest route is to read the client-side JavaScript and extract the hardcoded flag:

```bash
curl -sS https://make-me-dance-five.vercel.app/scrip.obf.js | grep -o "geek{[^}]*}"
```

This prints:

```text
geek{k0h_nah_m3}
```

## Flag
```text
geek{k0h_nah_m3}
```
