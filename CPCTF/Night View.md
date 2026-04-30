# Night View

## Flag

`CPCTF{381147502}`

## Writeup

The bridge is Rainbow Bridge.

The important part is not just identifying the bridge, but matching the rest of the skyline. In the challenge photo, the bridge is near the left side, there is a nearer midground building just to the right of it, and the large white tower cluster is much farther to the right.

That layout rules out the earlier Shiodome/Tsukuda guess. From candidates like `センチュリーパークタワー` or `シティフロントタワー`, the Tokyo Towers / Kachidoki cluster would sit only a few degrees to the right of Rainbow Bridge, so those towers would appear close to the bridge in the photo. They do not.

The Harumi / Triton Square towers fit much better. Using OSM coordinates:

- from `晴海ビュータワー1号棟`, Rainbow Bridge is the anchor view
- a nearer Harumi building appears about `+8°` to the right of the bridge
- the `The Tokyo Towers` / `Park Tower Kachidoki` group appears roughly `+25°` to `+32°` to the right

That matches the photo: bridge, then a nearer building overlapping it, then the big right-side tower cluster much farther right.

I also checked the nearby `晴海ビュータワー2号棟`. It is close, but from there the Triton office towers (`Office W/X`) would sit almost directly on top of Rainbow Bridge, which does not match the image as well. `1号棟` places those office towers farther left, which is more consistent with the challenge photo.

Finally, the OSM object is straightforward:

- `way 381147502`
- name: `晴海ビュータワー1号棟`
- address: `東京都中央区晴海1丁目6-1`

So the flag is:

`CPCTF{381147502}`

## Useful Commands

```bash
curl -s 'https://api.openstreetmap.org/api/0.6/way/381147502'

python3 - <<'PY'
import math
src = (35.6591946, 139.7839614)  # Harumi View Tower 1
rb  = (35.6363770, 139.7639279)  # Rainbow Bridge midpoint
def bearing(a,b):
    lat1,lon1=map(math.radians,a); lat2,lon2=map(math.radians,b)
    dlon=lon2-lon1
    x=math.sin(dlon)*math.cos(lat2)
    y=math.cos(lat1)*math.sin(lat2)-math.sin(lat1)*math.cos(lat2)*math.cos(dlon)
    return (math.degrees(math.atan2(x,y))+360)%360
print(bearing(src, rb))
PY
```

## References

- Challenge image: https://files.cpctf.space/Night_View/chal_55e58f3b2fb358e0733204a6acc7545150a321a5133281f89250094f28be5e82.jpg
- OSM way 381147502: https://api.openstreetmap.org/api/0.6/way/381147502
- Harumi Triton Square / nearby building geometry checked with OSM data around `晴海1丁目`
