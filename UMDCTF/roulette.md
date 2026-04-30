# roulette writeup

## Challenge

- Name: `roulette`
- Type: reverse engineering

## Summary

The binary does not print a flag directly. Instead, it checks whether the user submits a 106-byte "bet" that passes a custom validator. The accepted bet itself is the flag.

## Recon

The uploaded file is a stripped static x86-64 ELF. Important strings immediately show the structure:

- `lets go gambling!`
- `submit roulette number:`
- `>>> JACKPOT! You guessed the winning number.`
- `accepted`
- `rejected`

Disassembling around the entrypoint shows:

1. Input is read with `fgets`.
2. The trailing newline is removed.
3. The string is parsed as a base-10 integer.
4. A separate 106-byte validator runs before the binary prints `accepted`.

There is also a CRT-style helper function at `0x401cd0` and a VM-like bytecode interpreter driven by a table in `.rodata`.

## Main logic

The relevant code path does three things:

1. It derives a 32-bit seed for each of 27 rounds using hardcoded tables plus the CRT helper.
2. It initializes 8 VM registers:
   - `reg0 = seed`
   - `reg1 = rolling state`
   - `reg2 = round index`
   - the rest start at `0`
3. It interprets a tiny instruction stream whose handlers are:
   - copy register
   - xor register
   - xor immediate
   - add register
   - add immediate
   - multiply immediate
   - rotate-left immediate
   - rotate-left by register
   - shift-right immediate

At the end of each round, the VM result is XORed with a target table. That recovers one 32-bit little-endian chunk of the required input. The rolling state is then updated and fed into the next round.

Because every round is deterministic, the entire accepted input can be reconstructed offline without brute force.

## Recovering the flag

The reconstructed 106-byte accepted bet is:

`UMDCTF{I_R3ALLY-want-to-pl4y-the-p0werball,+but-my-d4d-said-no-so-im-b3tting-ill-win-on-POLYMARKETinstead}`

Running the binary with that exact string produces:

`accepted`

## Files

- [solve.py](/home/kali/ctf-orc/ctf/roulette/solve.py)
- [writeup.md](/home/kali/ctf-orc/ctf/roulette/writeup.md)
