# nuclear_codesa write-up

The provided `main` artifact is a stripped 64-bit PIE ELF. The program reads three positive integers `a,b,c`, enforces `a > b > c`, then spends most of its time in one helper at `0xfd60`.

## Key reverse result

By tracing that helper with `gdb` on small inputs:

- `fd60(1, 1) = e`
- `fd60(e, 1) = e^e`
- `fd60(1, e^e) = 0`
- `fd60(1, 2) = e - ln 2`

So the helper computes:

```text
fd60(x, y) = exp(x) - log(y)
```

The huge call chain in `main` is only `mov` + repeated `fd60` calls, so it can be symbolically traced. After simplification, the final left-hand side becomes

```text
L(a,b,c) =
(-8a^3 - 7a^2b + 19a^2c + 7ab^2 + abc - 10ac^2 - 8b^2c + 8bc^2 - c^3)
/
((a-c)(2a-b)(a+b-c))
```

and the right-hand side collapses to

```text
R = 2^-4096
```

The binary accepts when the final comparison sees the left expression as effectively zero, so solving the challenge reduces to finding an exact rational point on

```text
-8a^3 - 7a^2b + 19a^2c + 7ab^2 + abc - 10ac^2 - 8b^2c + 8bc^2 - c^3 = 0
```

## Exact solution

Using a rational-point search on the equivalent elliptic curve gave:

```text
a = 20260869859883222379931520298326390700152988332214525711323500132179943287700005601210288797153868533207131302477269470450828233936557
b = 19042526617180316524139256060457587477079898033904404413796747301621619442196095529358289579194164508926431543186710461288793130962534
c = 16792202595167632657252829598515092665938697948983181195334779924033054964579874761568657321835642556483381729386998074921169204991087
```

This triple satisfies `a > b > c > 0` and sends the local binary to the flag path.

## Remote solve

Sending that triple to `challs.umdctf.io:30301` returned:

```text
UMDCTF{ok_i_got_the_nuclear_codes_now_where_do_i_enter_them????}
```
