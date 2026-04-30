# Grass Toucher 10000 Writeup

## Challenge

- Name: `Grass Toucher 10000`
- Type: `OSINT / geolocation`
- Prompt: `Give the name of the county this picture was taken in.`

## Solve

The provided link was a short URL. I resolved it to a public SharePoint folder and enumerated the files through the SharePoint API. That yielded the original image:

- `I-LOVE-TOUCHING-GRASS.JPG`

I downloaded the image and checked its metadata with `exiftool`. There was no GPS data and no useful authoring metadata, so this had to be solved visually.

### What the image showed

The photo is a drone panorama of a rural, mountainous area with:

- a winding river on the left
- heavily forested ridgelines
- open fields and pasture
- a long light-roofed agricultural building / greenhouse area
- a pond
- a red barn / outbuilding

That terrain strongly suggested the Catskills / Appalachian northeast rather than flat farmland.

### Reverse-image and AI geolocation

I tried several reverse-image routes:

- Yandex image search
- Pinterest and real-estate matches from Yandex
- direct web searching for exact reposts

There was no exact duplicate, but the matches consistently clustered around northeastern rural mountain properties.

I then used a public geolocation service (`PlaceSpotter`) by creating a temporary account and uploading the image. The full image came back as:

- `Catskill Mountains, New York, USA`
- approx coords: `42.2146, -74.92`

Those coordinates reverse-geocode to:

- `Town of Hamden, Delaware County, New York`

I also checked map imagery around the Hamden / Delancey area and found the landscape style and farm/ridgeline layout were consistent with the photo. A likely supporting candidate in the same area was Berry Brook Farm / Back River Road in Hamden, which is also in Delaware County.

## Flag

`Delaware County`
