# Penetrable Tanish's Incident Report

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Log Analysis
- Artifact(s): server_stealth.log
- Prompt: The provided artifact was `server_stealth.log`, an Apache-style HTTP access log. The incident description hinted that normal-looking local subnet traffic may hide exfiltrated data.
- Final Flag: geek{here_c0m3es_sup3rman}

## Executive Summary
The obvious `/admin/config` token is a decoy. The real exfiltration channel is hidden in the response sizes of otherwise boring image requests from a single internal host.

## Solution Walkthrough
### Recon
Identify the artifact and sample it:

```bash
file server_stealth.log
wc -l server_stealth.log
head -n 20 server_stealth.log
```

The file is ASCII text with 1000 log entries. Most lines are noisy `/api/v1/data_*` requests from `10.0.0.0/24`.

There is also an obvious suspicious `/admin/config?token=...` value:

```bash
rg -n "admin/config|token" server_stealth.log
```

That token decodes from base64 to hex and then to:

```text
geek{6atman_is_dead}
```

This is a decoy, not the accepted flag.

### Finding the Covert Channel
The local host `192.168.1.100` is unusual because it only requests `/assets/image_*.png`, and it appears exactly 26 times:

```bash
awk '$1=="192.168.1.100"{print NR, $7, $9, $10}' server_stealth.log
awk '$1=="192.168.1.100"{print}' server_stealth.log | wc -l
```

There are 26 requests, matching the length of a plausible `geek{...}` flag. The response sizes are all around `1000`, which suggests the hidden byte is stored as:

```text
ASCII byte = response_size - 1000
```

Decode those response sizes in order:

```bash
awk '$1=="192.168.1.100"{print $10-1000}' server_stealth.log | awk '{printf "%c", $1} END{print ""}'
```

Output:

```text
geek{here_c0m3es_sup3rman}
```

## Flag
```text
geek{here_c0m3es_sup3rman}
```
