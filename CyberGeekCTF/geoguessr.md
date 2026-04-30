# GeoGuessr

## Challenge Overview
- Event: CyberGeek CTF
- Category: OSINT / Geolocation
- Prompt: Given a Google Street View 360 image, find the nearest city with population over 100,000.
- Final Flag: geek{Salalah}

## Executive Summary
The panorama metadata leaks the Street View ID. Resolving that ID yields Sarfayt, Oman, and a nearest-city check against population-filtered geodata gives Salalah.

## Solution Walkthrough
### Recon
Downloaded the image:

```bash
curl -L -o monsoon.jpg https://files.ctf7.com/media/challenge_attachments/monsoon.jpg
```

Basic metadata showed this was a Google Street View panorama and exposed the panorama ID:

```text
Projection Type: equirectangular
Software: Street View Download 360
Image ID: wk1y2YUtW9GD5WytedsxTg
Panorama ID: wk1y2YUtW9GD5WytedsxTg
```

### Location Recovery
Using the Street View panorama ID with the `streetlevel` helper library resolved the exact metadata:

```text
Pano ID: wk1y2YUtW9GD5WytedsxTg
Coordinates: 16.68254048732327, 53.11083663945605
Address: Sarfayt, Dhofar Governorate, Oman
Country: OM
Date: 2024-10
```

The visual clues also matched Dhofar, Oman: green monsoon-season mountains, arid cliffs, Omani-style buildings, and regional terrain.

### Nearest City Check
I used GeoNames `cities15000.txt`, filtered for population greater than 100,000, and computed haversine distance from the recovered panorama coordinates.

Closest qualifying cities:

```text
110.81 km  Salalah   OM  population 163140
488.74 km  Mukalla   YE  population 594951
```

Therefore the nearest city over 100k population is Salalah.

## Flag
```text
geek{Salalah}
```
