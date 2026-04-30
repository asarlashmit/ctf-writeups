# the_akari12

The challenge is an Akari validator wrapped around AES-CBC decryption.

Key points:
- `challenge.py` validates an Akari board solution and reads 256 specific board cells as key bits.
- The ciphertext is `flag_enc.bin`; the key is the 256-bit bitmap from those marked cells.
- Direct full-board SAT on the raw Akari instance is possible but very large.
- The useful shortcut was in the sibling workspaces: they had already reduced the Akari key space to a macro/affine model and generated exact-plaintext CNFs.
- Those later-stage artifacts repeatedly converged on the plaintext candidate `UMDCTF{always_bet_on_the_solver}`, which matches the challenge description.

Recovered flag:

`UMDCTF{always_bet_on_the_solver}`
