# exponentiall

Type: pyjail

Flag: `UMDCTF{exp0N3ntial_Grow7h_1s_too_f4st}`

## Bug

The service does:

```python
assert all(i==j for i,j in zip(sorted(Counter(c).values()),gen()))
assert all("~">=i not in "#'\" \t\n\r\x0c\x0b" for i in c)
exec(c)
```

So the input must:

- use only printable ASCII up to `~`
- avoid quotes, spaces, and whitespace
- have character frequencies exactly `1,2,4,...`

That means:

- with 16 distinct characters the total length is forced to `65535`
- using `()` is painful because `(` and `)` would have equal counts
- the easiest path is operator overloading on existing objects

## Loader

`site` builtins are available in script mode, so `exit` exists and its class is mutable.

Two key observations:

1. If a builtin is assigned to a binary special method, the operator calls it with the right-hand operand.
2. `exit<exit` can therefore be turned into `input(exit)`.

Stage one:

```python
exit.__class__.__lt__=input
c=exit<exit
exit.__class__.__lt__=exec
exit<c
```

After the first line, `exit<exit` reads an unrestricted second line into `c`.
After rebinding `__lt__` to `exec`, `exit<c` executes that second line.

The second stage is then simple:

```python
import glob;print(open(glob.glob("flag*")[0]).read());raise SystemExit
```

`raise SystemExit` is important because it exits before any of the padding executes.

## Padding

The loader uses 16 distinct characters, so the final payload must have counts:

- `n=1`
- `p=2`
- `u=4`
- `t=8`
- `< =16`
- `= =32`
- `; =64`
- `i=128`
- `x=256`
- `. =512`
- `e=1024`
- `_ =2048`
- `a=4096`
- `l=8192`
- `c=16384`
- `s=32768`

The trick is to append unreachable filler after `exit<c`.
Since the second stage exits immediately, the filler only needs to parse, not run.

Useful padding forms:

- `c.__class__...` to control `.`, `_`, and some `a/l/s/c`
- repeated bare identifiers like `ssss...` to add one character many times
- `c<c<...<c` to raise `<`
- `c=c=...=c` to raise `=`

I used a lower-AST version of the padding so the remote jail would accept it.
The generator is in [solve.py](/home/kali/ctf-orc/ctf/exponentiall/solve.py).
