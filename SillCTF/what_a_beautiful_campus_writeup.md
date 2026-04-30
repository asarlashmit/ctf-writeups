# What A Beautiful Campus!

Type: OSINT / Misc

## Summary

The short URL was fronted by Cloudflare, but `cloudscraper` resolved it to a public Penn State OneDrive folder. That folder contained a single file, `Image.jpg`.

The image was a screenshot of:

- 28 latitude/longitude pairs around Penn State University Park
- two single-digit columns aligned with those rows

The solve was:

1. Enumerate the OneDrive folder through SharePoint's guest session.
2. Download `Image.jpg`.
3. OCR the screenshot to recover the coordinate list and the two digit columns.
4. Resolve each coordinate to the exact campus label/building.
5. Treat the two digit columns as `(word_index, letter_index)` for each resolved name.

That extraction produced:

```text
sillyCTFthatsalotofbuildings
```

Using the same flag format as the other challenge in this repo, the flag is:

```text
sillyCTF{thatsalotofbuildings}
```

## Recon

The short link redirected to a Penn State OneDrive share:

```text
https://www.shorturl.at/jxGia
-> https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/mqy5254_psu_edu/IgDkJ7t_AswYS49aWKHh6gEwAcoVDI70XBFNUmNr45DTJiA?e=oHCsJL
```

I used the anonymous SharePoint session cookie from that redirect to list the shared folder:

```bash
python - <<'PY'
import requests

share='https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/mqy5254_psu_edu/IgDkJ7t_AswYS49aWKHh6gEwAcoVDI70XBFNUmNr45DTJiA?e=oHCsJL'
s=requests.Session()
s.headers['Accept']='application/json;odata=verbose'
s.get(share, allow_redirects=True, timeout=60)

url="https://pennstateoffice365-my.sharepoint.com/personal/mqy5254_psu_edu/_api/web/GetFolderByServerRelativeUrl('/personal/mqy5254_psu_edu/Documents/campus')/Files"
print(s.get(url, timeout=60).text)
PY
```

That showed a single file:

```text
Image.jpg
```

## OCR

Local OCR was noisy, but OCR plus manual cleanup recovered 28 coordinate rows and two aligned digit columns.

The important observation was that the digits were not 2-digit numbers. They were two separate columns, so each row had a pair like:

```text
(1,7), (1,2), (1,6), ...
```

Those pairs are `(word_index, letter_index)`.

## Mapping Coordinates To Campus Labels

For the building-heavy rows, the most reliable source was Penn State's campus polygon layer:

```text
https://mapservices.pasda.psu.edu/server/rest/services/pasda/PSU_Campus/MapServer/1
```

For a few rows, the public-facing label mattered more than the campus asset name. The useful label variants were:

- `Student Fitness Center` instead of just `Rec Hall`
- `Nittany Lion Softball Park` / `Beard Field at Nittany Lion Softball Park`
- `Pattee and Paterno Library`
- `Agricultural Sciences and Industries Building`
- `Weston Community Center`
- `Panofsky Hall`
- `Thomas Building`

Once the names were normalized to the labels users would actually see on a map, the pairs decoded cleanly.

Examples:

- `Patterson Building` with `(1,7)` -> `s`
- `Pinchot Hall` with `(1,2)` -> `i`
- `Steidle Building` with `(1,6)` -> `l`
- `Shulze Hall` with `(2,4)` -> `l`
- `Stuckeman Family Building` with `(2,6)` -> `y`

That starts with:

```text
silly...
```

Continuing this across all 28 rows gives:

```text
sillyCTFthatsalotofbuildings
```

## Flag

```text
sillyCTF{thatsalotofbuildings}
```
