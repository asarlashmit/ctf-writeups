# Iris out

## Flag

`CPCTF{35674_139765}`

## Summary

The performance location is the abandoned `首都高速八重洲線 / Yaesu Route` approach to `西銀座料金所 / 西銀座乗継所` next to `東京交通会館` and the `有楽町/銀座` Uniqlo side.

The white circular platform that appears at the end is on the short elevated segment between the toll area and the visible curve. A good center estimate is:

- Latitude: `35.6743`
- Longitude: `139.7648`

After multiplying by `1000` and rounding to integers:

- `35.6743 * 1000 -> 35674`
- `139.7648 * 1000 -> 139765`

So the flag is:

- `CPCTF{35674_139765}`

## How It Was Solved

1. General location

   Multiple public writeups/articles about the NHK performance identify the filming location as the old `KK線` / `首都高速八重洲線` area around `西銀座JCT`.

   - `tvidealife.com` says the phone-booth part was near the former `西銀座料金所`.
   - `domin7.com` cites a witness report placing the shoot between `有楽町ユニクロ` and `東京交通会館`.

2. End-stage position

   A Note article with screenshots from the performance shows:

   - the toll structure,
   - the nearby elevated road segment,
   - the final white circular platform.

   Those screenshots place the end platform on the abandoned Yaesu Route approach side of the toll area, not farther down the KK loop.

3. Map confirmation

   OpenStreetMap road geometry for the abandoned Yaesu Route in that exact area shows the matching elevated segment and the `西銀座乗継所` toll-booth nodes.

   The matching road segment is the one around:

   - `35.6739149, 139.7645303`
   - `35.6746323, 139.7650450`

   The platform center falls roughly in that bucket, so `35.6743, 139.7648` is a consistent estimate.

## Useful Sources

- Official performance video: `https://www.youtube.com/watch?v=MJ70B_cwnPk`
- Location discussion: `https://tvidealife.com/nhk-kouhaku-filming-location-2/`
- Witness-based location narrowing: `https://domin7.com/kenshi-yonezu-kohaku2025-location/`
- Screenshot set: `https://note.com/same_kudu1254/n/n2962b92fde91`
- OSM road geometry:
  - `https://api.openstreetmap.org/api/0.6/map.json?bbox=139.7630,35.6733,139.7658,35.6756`
  - `https://api.openstreetmap.org/api/0.6/way/24402126/full.json`

