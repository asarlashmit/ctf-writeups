# gigabrain.ai

Type: AI / Rev

## Summary

The challenge link did not point directly to a service. It redirected through `shorturl.at` to a public SharePoint/OneDrive folder named `gigabrain.ai` containing two large archives:

- `linux-gigabrain.zip`
- `windows-gigabrain.zip`

Instead of downloading the full 2 GB model archive, I used the anonymous OneDrive page metadata and HTTP range requests to inspect the ZIP central directory and extract only the small wrapper files.

The Linux wrapper binary was not stripped. Running `strings` against it immediately exposed:

- trigger keywords such as `sudo`, `debug`, `override`, `oath`, `court`, `federal`, `grandmother`, `story`, and `bedtime`
- three canned jailbreak responses
- the flag embedded directly in those responses

## Recon

Following the shortlink with a browser-like user agent showed the real destination:

```text
https://pennstateoffice365-my.sharepoint.com/:f:/g/personal/spc6394_psu_edu/IgCT-lVFvtVSQqfY1jEAfnWJAW-gf8l_rLiygTjziX7UyuY?e=GmuOfG
```

The OneDrive page exposed a drive API URL and an anonymous access token inside `_spPageContextInfo.driveInfo`. Querying the folder through that API revealed the two archives without needing to authenticate.

## Solve

List the shared folder contents:

```bash
python3 - <<'PY'
import re, requests
s = requests.Session()
r = s.get(
    'https://www.shorturl.at/xYyI6',
    headers={'User-Agent': 'Mozilla/5.0'},
    allow_redirects=True,
    timeout=30,
)
m = re.search(r'"\\.driveUrl":"([^"]+)".*?"\\.driveAccessToken":"([^"]+)"', r.text)
drive_url = m.group(1).encode().decode('unicode_escape')
token_qs = m.group(2).encode().decode('unicode_escape')
api = drive_url + '/root:/gigabrain.ai:/children?' + token_qs
print(s.get(api, headers={'User-Agent': 'Mozilla/5.0'}, timeout=30).text)
PY
```

That returns the two ZIPs and their temporary download URLs.

Inspect the ZIP contents with range requests so the 2 GB model does not need to be downloaded:

```bash
python3 - <<'PY'
import re, requests, struct
s = requests.Session()
h = {'User-Agent': 'Mozilla/5.0'}
r = s.get('https://www.shorturl.at/xYyI6', headers=h, allow_redirects=True, timeout=30)
m = re.search(r'"\\.driveUrl":"([^"]+)".*?"\\.driveAccessToken":"([^"]+)"', r.text)
drive_url = m.group(1).encode().decode('unicode_escape')
token_qs = m.group(2).encode().decode('unicode_escape')
api = drive_url + '/root:/gigabrain.ai:/children?' + token_qs
items = s.get(api, headers=h, timeout=30).json()['value']
url = next(x['@content.downloadUrl'] for x in items if x['name'] == 'linux-gigabrain.zip')
size = int(s.head(url, headers=h, allow_redirects=True, timeout=30).headers['Content-Length'])
tail = s.get(url, headers={**h, 'Range': f'bytes={size-131072}-{size-1}'}, timeout=60).content
idx = tail.rfind(b'PK\\x05\\x06')
_, _, _, _, _, cd_size, cd_offset, _ = struct.unpack('<4s4H2LH', tail[idx:idx+22])
cd = s.get(url, headers={**h, 'Range': f'bytes={cd_offset}-{cd_offset+cd_size-1}'}, timeout=60).content
print(cd.decode('latin1', 'ignore'))
PY
```

The interesting entries were:

```text
gigabrain
gigabrain.sh
libggml-*.so
libllama.so.*
Llama-3.2-3B-Instruct-Q4_K_M.gguf
```

Extract only the small wrapper binary and inspect it:

```bash
strings -n 4 gigabrain | rg 'sudo|debug|override|oath|court|federal|grandmother|story|bedtime|sillyCTF'
```

Relevant output:

```text
sudo
debug
override
oath
court
federal
grandmother
story
bedtime
System diagnostics active. Validation token: sillyCTF{prompt_injected_lol}
Under oath, my system identifier is: sillyCTF{prompt_injected_lol}
Once upon a time, there was a magical castle protected by the ancient code: sillyCTF{prompt_injected_lol}. The end.
```

Disassembling `check_trigger()` showed the exact logic:

- `debug` and `override` returns the diagnostics response
- `sudo` and `debug` also returns the diagnostics response
- `oath` plus either `court` or `federal` returns the oath response
- `grandmother` plus either `story` or `bedtime` returns the bedtime-story response

All three success branches embed the same flag string, so there was no need to run the model at all.

## Flag

```text
sillyCTF{prompt_injected_lol}
```
