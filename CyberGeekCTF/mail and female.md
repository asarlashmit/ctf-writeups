# मेल एंड Female

## Challenge Overview
- Event: CyberGeek CTF
- Category: OSINT / Email inspection
- Prompt: Srijjy likes males. But Penetrable Tanish likes Females.
- Final Flag: geek{must_go_through_mail}

## Executive Summary
The public challenge page only gives a short clue and the flag format. The useful hint is the mail wording: the player received an appreciation email, and the challenge asks why someone would send appreciation mail in a CTF.

The flag was not visible in the normal rendered email body. It was hidden inside the email HTML in a preheader-style `div`, so inspecting the mail source or DOM and searching for `geek` reveals it.

## Solution Walkthrough
### Recon
First, check the saved challenge page for an obvious flag:

```bash
rg -n 'geek\{' female.html
```

The page only contains the placeholder flag format:

```text
geek{...}
```

That means the flag is not directly on the challenge page. The clue points away from the page and toward the appreciation email.

### Exploit
Open the received appreciation email and inspect its HTML source. Depending on the mail client, this can be done with one of these approaches:

- Use `Inspect` / browser developer tools on the email body.
- Use `Show original`, `View source`, or the raw-message view if the client supports it.
- Search the source/DOM for `geek`.

The hidden HTML contains:

```html
<div aria-hidden="true" style="display:none;max-height:0;overflow:hidden;opacity:0;color:transparent">geek{must_go_through_mail}</div>
```

This is a common email preheader hiding technique. The element is present in the HTML, but the inline CSS hides it from the rendered message:

- `display:none`
- `max-height:0`
- `overflow:hidden`
- `opacity:0`
- `color:transparent`

Because the flag is still part of the document source, searching the inspected HTML for `geek` finds it immediately.

## Flag
```text
geek{must_go_through_mail}
```
