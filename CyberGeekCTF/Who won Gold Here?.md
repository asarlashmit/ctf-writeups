# Who won Gold Here?

## Challenge Overview
- Event: CyberGeek CTF
- Category: OSINT / Geolocation / Sports
- Prompt: Resolve `https://maps.app.goo.gl/DzAnUBdUmCsFajRA9` and identify who won gold there.
- Final Flag: geek{palmer}

## Executive Summary
The Google Maps short link resolves to Ariake Urban Sports Park in Tokyo. Matching that venue to the relevant Olympic event identifies Keegan Palmer as the gold medalist.

## Solution Walkthrough
### Solution
The challenge only provides a Google Maps short link and asks who won gold at that place.

First, I expanded the Maps URL:

```bash
curl -sSL -D - 'https://maps.app.goo.gl/DzAnUBdUmCsFajRA9' -o /tmp/maps.html
```

The redirect location identifies the place as:

```text
livedoor URBAN SPORTS PARK (Ariake Urban Sports Park)
```

This is in Ariake, Tokyo. The important keyword is the older venue name, **Ariake Urban Sports Park**.

Searching that venue shows that it was built for the **Tokyo 2020 Summer Olympics** and hosted urban sports events, including skateboarding and BMX. Since the question says "won Gold Here", the next step is to look for Olympic gold medalists from events held at Ariake Urban Sports Park.

The relevant result is the Tokyo 2020 **Skateboarding Men's Park** event. It took place at **Ariake Urban Sports Park** on **5 August 2021**.

The final results list:

```text
Gold: Keegan Palmer
Silver: Pedro Barros
Bronze: Cory Juneau
```

Keegan Palmer won the event for Australia with a best score of `95.83`.

## Flag
```text
geek{palmer}
```

## References and Notes
### References
- Google Maps target: `livedoor URBAN SPORTS PARK (Ariake Urban Sports Park)`
- Ariake Urban Sports Park venue information: https://en.wikipedia.org/wiki/Ariake_Urban_Sports_Park
- Tokyo 2020 Men's Park result: https://en.wikipedia.org/wiki/Skateboarding_at_the_2020_Summer_Olympics_%E2%80%93_Men%27s_park
- NBC Olympics result summary: https://www.nbcolympics.com/news/aussie-teen-keegan-palmer-takes-historic-gold-usas-cory-juneau-nabs-bronze-skateboard-park
