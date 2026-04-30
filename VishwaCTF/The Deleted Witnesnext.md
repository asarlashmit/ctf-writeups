# The Deleted Witnesnext Writeup

## Challenge Type
Forensics / mobile app data recovery

## Initial Recon
The archive contains a small Android-style extraction:

- `android_extraction/Download/note.txt`
- `android_extraction/DCIM/family_photo.jpg`
- `android_extraction/DCIM/trip_photo.jpg`
- `android_extraction/cache/.thumbnails/img_cache_1.tmp`
- `android_extraction/apps/com.legitapp.browser/databases/browser.db`

There are several decoys:

- `family_photo.jpg` EXIF comment: `VishwaCTF{fake_flag_in_exif}`
- `trip_photo.jpg` EXIF comment: `Th1s_1s_N0t_Th3_K3y`
- `browser.db` search row: `VishwaCTF{br0ws3r_h1st0ry_1s_n0t_3n0ugh}`
- decrypted app message: `VishwaCTF{wr0ng_try_4ga1n}`

That makes it clear the live browser data is only a hint, not the answer.

## Useful Clue
`trip_photo.jpg` has this EXIF `User Comment`:

```text
s3cr3t5_aR3_B3tt3r_k33pT_s3cR3t5
```

The note says:

```text
Remember:
- Delete the app before giving phone
- They can't recover anything once its gone
- The key is safe, nobody will find it
```

This strongly suggests an encrypted deleted-app artifact.

## Real Target
`android_extraction/cache/.thumbnails/img_cache_1.tmp` is not a normal thumbnail cache. It is a 16 KiB high-entropy blob and decrypts correctly as an SQLCipher database with the EXIF `User Comment` as the key.

List the tables:

```bash
sqlcipher extracted/android_extraction/cache/.thumbnails/img_cache_1.tmp <<'EOF'
PRAGMA key='s3cr3t5_aR3_B3tt3r_k33pT_s3cR3t5';
SELECT name FROM sqlite_master WHERE type='table';
EOF
```

This reveals:

```text
messages
users
media
```

## Extracting the Fragments
Dump the rows:

```bash
sqlcipher extracted/android_extraction/cache/.thumbnails/img_cache_1.tmp <<'EOF'
.mode list
.separator |
PRAGMA key='s3cr3t5_aR3_B3tt3r_k33pT_s3cR3t5';
SELECT id,username,token FROM users;
SELECT id,sender,receiver,body,timestamp,deleted FROM messages;
SELECT id,filename,data,timestamp FROM media;
EOF
```

Relevant values:

```text
1|alice|V9pRl90SDNfYXBQTDFDNFRpT25faTVfYzBtUHIwTW
1|alice|bob|VmlzaHdhQ1RGe0IzM19pVF80bkRyMGlEXzBSX2kwN|1710000001|1
1|selfie.jpg|lzM0RfWTBVX2FyM19EMG4zX0Ywcl9ZMHJfTDFGM30=|1710000005
```

The deleted message body, Alice's token, and the media blob are three Base64 fragments.

## Reconstructing the Flag
Concatenate them in this order:

1. deleted `messages.body`
2. `users.token`
3. `media.data`

Decode:

```bash
python3 - <<'PY'
import base64
s = (
    'VmlzaHdhQ1RGe0IzM19pVF80bkRyMGlEXzBSX2kwN'
    'V9pRl90SDNfYXBQTDFDNFRpT25faTVfYzBtUHIwTW'
    'lzM0RfWTBVX2FyM19EMG4zX0Ywcl9ZMHJfTDFGM30='
)
print(base64.b64decode(s).decode())
PY
```

Output:

```text
VishwaCTF{B33_iT_4nDr0iD_0R_i05_iF_tH3_apPL1C4TiOn_i5_c0mPr0Mis3D_Y0U_ar3_D0n3_F0r_Y0r_L1F3}
```

## Flag
`VishwaCTF{B33_iT_4nDr0iD_0R_i05_iF_tH3_apPL1C4TiOn_i5_c0mPr0Mis3D_Y0U_ar3_D0n3_F0r_Y0r_L1F3}`
