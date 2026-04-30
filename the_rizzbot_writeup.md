# The Rizzbot

Type: Steganography

## Flag

`sillyCTF{thatsalotofbuildings}`

## Solve

The provided short URL was protected by Cloudflare in a plain `curl` session, but a browser view showed it redirected to a Penn State OneDrive folder named `rizzbot` containing a single file:

```text
skibidirizzbot.mp3
```

The MP3 itself had a useful hint in its metadata:

```text
TIT3: RizzToTheMax
TIT2: RIZZBOT100000
artist: SIllyCTF
WM/Mood: ROBOT
```

That pointed at a nonstandard audio-to-visual decode path rather than a simple appended-file trick. The recovered local stego artifacts for this solve ended up as three cleaned table images:

- `top_prep.png`
- `mid_prep.png`
- `bot_prep.png`

Together they contain the real payload: a 28-row screenshot with

- Penn State University Park latitude/longitude pairs
- a `W` column
- an `L` column

Those last two columns are `word_index` and `letter_index`.

Examples from the recovered table:

```text
40.800306 -77.864929   1   7
40.805706 -77.863373   1   2
40.795194 -77.865206   1   6
40.801166 -77.857651   2   4
40.801121 -77.866701   2   6
...
40.803458 -77.863758   4   5
```

At that point the problem becomes the same Penn State campus-label extraction used elsewhere in this workspace. Resolve each coordinate to the campus-facing building/location label, then take `(word_index, letter_index)` from that label.

Examples:

- `Patterson Building` with `(1,7)` -> `s`
- `Pinchot Hall` with `(1,2)` -> `i`
- `Steidle Building` with `(1,6)` -> `l`
- `Shulze Hall` with `(2,4)` -> `l`
- `Stuckeman Family Building` with `(2,6)` -> `y`

Continuing across all 28 rows yields:

```text
sillyCTFthatsalotofbuildings
```

Applying the normal flag format gives:

```text
sillyCTF{thatsalotofbuildings}
```

## Notes

- The user-supplied type said `pwn`, but the local challenge metadata identifies `The Rizzbot` as `Steganography`.
- For the campus-name step, the most reliable public data source was Penn State's campus map layer:

```text
https://mapservices.pasda.psu.edu/server/rest/services/pasda/PSU_Campus/MapServer/1
```
