# hades-groupret Write-up

## Challenge Type

OSINT / Telegram export attribution.

## Summary

The export was a standard Telegram JSON dump for `Hades Group` with 400 messages. Anonymous owner/admin posts were sent as `channel28740651`. Most text posts were operational rules, but the anonymous channel account also posted one sticker:

- Sticker set: `styx_reaction_pack`
- File reference: `stickers/styx_reaction_pack_001.webp`

That sticker set was the key attribution artifact.

## Solve Path

1. Profiled `hades_export.json` and isolated anonymous posts from `channel28740651`.
2. Found the only anonymous-admin media artifact: `styx_reaction_pack`.
3. Queried the challenge-provided sticker intelligence bot for that sticker set and got creator UID `7816442093`.
4. Queried the provided leak/intel bots for UID `7816442093`, then followed the alias chain:
   - `@zeus_archive`
   - `@thanatos_signal`
   - `@kerberos_spine`
5. The profile observer bot linked `@thanatos_signal` to German phone `+49 160 5550 7318`.
6. Leak bots resolved that phone to `Niklas Hofmann`, linked back to UID `7816442093`.
7. The generic country document bot, set to Germany, returned:
   - Record: `REC-9305174`

## Flag

`UMDCTF{REC-9305174}`

## Verification

Submitted `UMDCTF{REC-9305174}` to the UMDCTF platform for `hades-group`; response was `goodFlag`.
