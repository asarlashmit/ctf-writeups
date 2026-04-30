# Dead Dropnew Writeup

## Challenge

- Name: `Dead Dropnew`
- Type: `forensics / network`
- Flag format: `VishwaCTF{}`

## Recon

The attachment contains a 68-packet `pcapng` capture:

```bash
capinfos dead_drop.pcapng
```

Important observations:

- Most traffic is DNS between `10.0.1.114` and `10.0.0.1`
- There is a short TCP/TLS conversation with `185.220.101.47`
- The very last packet, frame `68`, is a standalone UDP packet to `185.220.101.47`

The suspicious DNS queries are the ones under `r3s.io`:

```text
KZUX
G2DX
MFBV
IRT3
MRXH
GX3U
OVXG
C3X8EMJQ
4ZLM
L5ZD
G5RT
GRWD
GZC7
MJ4V
65DU
NRPW
C3TEL52GS3LJNZTX2
```

## DNS Exfil Reconstruction

These labels are an uppercase Base32-looking stream. Concatenating every label fails because `C3X8EMJQ` contains an invalid Base32 character (`8`) and breaks the decode.

That 8-character label is also the only outlier in the middle of what is otherwise:

- repeated 4-character chunks
- one final longer remainder chunk

Dropping the lone control/outlier label gives:

```text
KZUXG2DXMFBVIRT3MRXHGX3UOVXG4ZLML5ZDG5RTGRWDGZC7MJ4V65DUNRPWC3TEL52GS3LJNZTX2
```

Decoding that string as Base32 produces the payload directly:

```text
VishwaCTF{dns_tunnel_r3v34l3d_by_ttl_and_timing}
```

Minimal reproduction:

```python
import base64
s = "KZUXG2DXMFBVIRT3MRXHGX3UOVXG4ZLML5ZDG5RTGRWDGZC7MJ4V65DUNRPWC3TEL52GS3LJNZTX2"
print(base64.b32decode(s + "=" * ((8 - len(s) % 8) % 8)).decode())
```

## Final UDP Signal

Frame `68` is:

- `10.0.1.114:49227 -> 185.220.101.47:53412`
- UDP
- TTL `79`
- sent `0.030000000` seconds after frame `67`

The packet body (`00 00 4f 4b 4f 4b 4f 4b 4f 4b`) is a decoy. The useful signal is in the packet metadata rather than the payload, which aligns with the recovered flag phrase: `ttl_and_timing`.

## Flag

```text
VishwaCTF{dns_tunnel_r3v34l3d_by_ttl_and_timing}
```
