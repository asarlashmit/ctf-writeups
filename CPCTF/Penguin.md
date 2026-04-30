# Penguin

## Challenge

- Name: `Penguin`
- Type: `OSINT / image analysis`

## Goal

The challenge asks for the individual number of the penguin in the upper right of the attached image.

## Recon

The useful part of the image is the flipper band on the target penguin. The best local crop was:

- `crops/orig_band_enh_3600_1250.png`

That crop shows a band with:

- a white base strap
- an orange/yellow tab
- a green tab
- a white tab
- a blue tab
- the buckle

The hint points toward the same penguin group as emperor penguins, and toward Nagoya Port Aquarium / Adventure World. The important reference turned out to be the Nagoya Port Aquarium flipper-band guide:

- `band_guide2.pdf`
- `band_guide.pdf`

Those references confirm the decoding rules:

- `black=0`
- `red=1`
- `blue=2`
- `yellow=3`
- `green=4`
- `white=5`
- read from `せなか側` to `おなか側`
- if the visible band has 3 tabs, start from the tens digit
- the ones digit is the sum of the last 2 tabs
- if the base band is white, add `50`

## Decode

From the target crop, the band matches the right-flipper orientation and the visible order is:

- `yellow`
- `green`
- `white`
- `blue`

The top tab looks orange in the underwater photo, but its hue is much closer to yellow/orange than to red, so it should be treated as the official `yellow` tab.

So the number is:

- hundreds: `yellow = 3`
- tens: `green = 4`
- ones: `white + blue = 5 + 2 = 7`
- white base band: `+ 50`

Therefore:

`347 + 50 = 397`

## Flag

`CPCTF{397}`
