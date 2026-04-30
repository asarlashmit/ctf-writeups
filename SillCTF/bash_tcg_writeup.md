# Bash TCG Writeup

Challenge: `Bash TCG`

Category: `Pwn`

Flag: `sillyCTF{Thats_a_wEiRd_Name}`

## Summary

The binary is a 32-bit non-PIE ELF with partial RELRO and a hidden `backdoor()`:

```c
void backdoor() {
    execv("/bin/sh", NULL);
}
```

The bug is in `print_card_stats()`. Card nicknames are copied onto a stack buffer and then passed directly to `printf()` as the format string:

```c
memcpy(localbuf, card->nickname, 0x63);
printf("Nickname: ");
printf(localbuf);
putchar('\n');
```

Only MEGA cards are interesting, because `rename_card()` uses `strcpy()` for them and allows a long nickname:

```c
if (card->is_mega)
    strcpy(card->nickname, buf);
else
    strncpy(card->nickname, buf, 10);
```

So the exploit plan is:

1. Open packs until a MEGA card appears.
2. Rename it with a format-string payload.
3. Use `print_card_stats()` to trigger the bug.

## Recon

Useful symbols:

- `print_card_stats = 0x08049338`
- `rename_card = 0x0804980d`
- `backdoor = 0x08049ba2`

Important GOT entries:

- `puts@got = 0x0804c038`
- `atoi@got = 0x0804c058`

By probing the format string with `AAAABBBBCCCCDDDD.%7$p.%8$p...`, the first four 32-bit words of the nickname buffer were confirmed at arguments `7`, `8`, `9`, and `10`.

Later stack probing showed:

- `%34$p` = saved `ebp` of the caller
- `%35$p` = saved return address of `print_card_stats()` (`0x8049b57`)

That was the cleanest control target.

## Exploit

My first idea was GOT overwrite:

- overwrite `puts@got` with `backdoor`
- trigger the `puts()` call immediately after the vulnerable `printf(localbuf)`

This reached control flow, but it was less reliable because it hijacked execution in the middle of output.

The stable exploit was to overwrite the saved return address of `print_card_stats()` on the stack.

### 1. Leak the caller frame pointer

Rename the MEGA card to:

```text
LEAK.%34$p.END
```

Then print the card stats and parse the leaked saved `ebp`.

### 2. Compute the saved return-address slot

At the call site in `main`, the stack layout makes the saved return address for `print_card_stats()` live at:

```text
ret_slot = saved_ebp - 0x4c
```

### 3. Overwrite `ret_slot` with `backdoor`

I used four `%hhn` writes to keep the output short and reliable.

For `backdoor = 0x08049ba2`, the bytes are:

- `0xa2`
- `0x9b`
- `0x04`
- `0x08`

These were written to `ret_slot + 0..3` in ascending-byte order using positional arguments `7` through `10`.

### 4. Trigger return into the shell

Call `print_card_stats()` again with the overwrite payload. The function finishes normally, then returns into `backdoor()`, which executes `/bin/sh`.

From the shell:

```sh
cat /chall/flag.txt
```

Output:

```text
sillyCTF{Thats_a_wEiRd_Name}
```

## Solve Script

The exploit script used for the final solve is:

- `solve_bash_tcg.py`

It automates:

1. farming a MEGA card
2. leaking `%34$p`
3. computing the stack return slot
4. overwriting it with `backdoor`
5. reading `/chall/flag.txt`
