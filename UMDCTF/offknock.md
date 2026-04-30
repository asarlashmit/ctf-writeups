# offknock

The DNS server checks the raw bytes at `request[12:30]` for:

`04 flag 06 market 03 pol c8 b3`

That sequence is not a valid normal encoding of `flag.market.polȳ`. On the wire, `c8 b3` is treated as a DNS compression pointer, not literal UTF-8.

The intended bypass is to abuse the remote parser accepting a forward compression pointer:

1. Send a DNS TXT query whose QNAME starts with `\x04flag\x06market\x03pol\xc8\xb3`.
2. `c8 b3` becomes a pointer to offset `0x08b3`.
3. Pad the packet until offset `0x08b3`, then place `\x00` there so the pointed name ends at the root.
4. The raw-byte check passes, while the parsed question becomes `flag.market.pol.` with type `TXT`.
5. The server returns a TXT record containing the flag.

Flag: `UMDCTF{175_4_107_34513r_70_M4K3_KN0CK0FF5_N0W}`
