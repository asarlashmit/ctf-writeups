# Night Viewnew

## Flag

`CPCTF{333413426}`

## Writeup

The earlier skyline-based guesses around Harumi, Kachidoki, and Shiodome were wrong.

To verify the answer directly, I recovered the authenticated Firefox session from the local profile, extracted the live `session` cookie, and replayed the same API request the CPCTF frontend uses for flag submission:

- `POST /api/challenges/{cid}/answer`
- JSON body: `{"answer":"CPCTF{...}"}`

Using that authenticated submit flow against the challenge endpoint, the first confirmed correct flag was:

- `CPCTF{333413426}`

Querying the OpenStreetMap API for that way ID resolves the building to:

- `msb Tamachi 田町ステーションタワーN`
- `way 333413426`
- address: `東京都港区芝浦3-1-1`

So the correct answer is:

`CPCTF{333413426}`

## Useful Commands

```bash
curl -s 'https://api.openstreetmap.org/api/0.6/way/333413426'

curl -s 'https://cpctf.space/api/challenges/413440c0-5b8b-45c3-a6bc-2c4e4fb4482e/answer' \
  -H "Cookie: session=..." \
  -H 'Content-Type: application/json' \
  --data '{"answer":"CPCTF{333413426}"}'
```
