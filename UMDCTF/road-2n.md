# road-2n Writeup

## Challenge

Given a cropped traffic camera screenshot, identify the intersection shown. The real camera feed includes the intersection label at the bottom, and the flag is that label wrapped in `UMDCTF{}`.

## Recon

- Downloaded `road_2.png` from the challenge URL.
- Verified it was a plain PNG image with no useful embedded metadata.
- Cropped and enlarged visible clue areas.
- The key visual clue was not a railroad marking; it was a pair of protected bike lanes with white flex posts at a signalized intersection.

## Solve

The protected bike lanes, rural/suburban edge, traffic signal layout, and field next to the road matched Montgomery County / MD 355 traffic cameras. The WeatherBug/TrafficLand listing for camera `2205` is:

`Frederick Rd (MD-355) @ Clarksburg Rd (MD-121)`

Fetching the live TrafficLand image for that camera showed the on-video overlay:

```text
MD355&CLARKSBURG RD
CAM 121
```

The challenge asks for the intersection text, so the flag uses only the first overlay line.

## Flag

`UMDCTF{MD355&CLARKSBURG RD}`
