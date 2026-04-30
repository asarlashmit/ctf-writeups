# Bytecode is coming

Type: `rev`

Flag: `Kaal{7h15_c0d3_15_4n_4b50lu73_m355_4nd_1_5t1ll_d0n7_kn0w_h0w_17_w0rk5}`

## Summary

The challenge ships a Python 3.14 `.pyc` file. Running it directly asks for a file path and validates the file byte-by-byte against a hidden transform. The intended solve path is to recover the validation tables from the bytecode, invert the per-byte transformation, reconstruct the correct file, then read the flag from the restored image.

## Recon

The archive only contained `Kaal.pyc`.

Useful observations:

- The magic bytes match Python 3.14 bytecode.
- The module defines helpers `_S`, `_U`, `_UI`, `_rot`, and `_main`.
- `_main()` loads a user-supplied file, enforces length `74463`, and checks each byte against hidden tables:
  - `_K`, `_A`, `_B`, `_C`, `_D`, `_R`, `_I`, `_E`

I loaded the code object with the bundled Python 3.14 interpreter:

```bash
./Python-3.14.3/python - <<'PY'
import marshal, pathlib
co = marshal.loads(pathlib.Path('Kaal.pyc').read_bytes()[16:])
print(co.co_name, len(co.co_code), co.co_names)
PY
```

Then executed it safely with `__name__ != "__main__"` to materialize the tables without triggering the input prompt:

```bash
./Python-3.14.3/python - <<'PY'
import marshal, pathlib
co = marshal.loads(pathlib.Path('Kaal.pyc').read_bytes()[16:])
ns = {'__name__': 'not_main'}
exec(co, ns)
for k in ['_target','_K','_E','_I','_A','_B','_C','_D','_R']:
    v = ns[k]
    print(k, len(v) if hasattr(v, '__len__') else v)
PY
```

## Validation Logic

For each byte index `i`, the program computes:

```python
exp = _E[_I[i]]
if _K[i] == 0:
    got = (((byte ^ _A[i]) + _B[i]) & 0xff) ^ _C[i]
else:
    got = rol8(byte, _R[i]) ^ _D[i]
```

So every byte can be inverted independently.

## Reconstructing the File

I inverted both branches:

```python
if _K[i] == 0:
    byte = (((exp ^ _C[i]) - _B[i]) & 0xff) ^ _A[i]
else:
    t = exp ^ _D[i]
    r = _R[i]
    byte = ((t >> r) | ((t << (8 - r)) & 0xff)) & 0xff
```

Recovery script:

```bash
./Python-3.14.3/python - <<'PY'
import marshal, pathlib

co = marshal.loads(pathlib.Path('Kaal.pyc').read_bytes()[16:])
ns = {'__name__': 'not_main'}
exec(co, ns)

N = ns['_target']
K, A, B, C, R, D, I, E = [ns[x] for x in ['_K','_A','_B','_C','_R','_D','_I','_E']]

out = bytearray(N)
for i in range(N):
    exp = E[I[i]]
    if K[i] == 0:
        out[i] = (((exp ^ C[i]) - B[i]) & 0xff) ^ A[i]
    else:
        t = exp ^ D[i]
        r = R[i]
        out[i] = ((t >> r) | ((t << (8 - r)) & 0xff)) & 0xff

pathlib.Path('recovered.bin').write_bytes(out)
print(out[:16].hex())
PY
```

The header immediately showed a JPEG:

```text
ffd8ffe000104a464946000101010060
```

And `file recovered.bin` reported:

```text
JPEG image data, JFIF standard 1.01, 800x800
```

## Extracting the Flag

The restored file was a slanted image containing the flag text. Standard OCR on the full image was poor, so I used `rapidocr_onnxruntime` and also rectified the detected text lines to horizontal strips.

The recovered flag text was:

```text
Kaal{7h15_c0d3_15_4n_4b50lu73_m355_4nd_1_5t1ll_d0n7_kn0w_h0w_17_w0rk5}
```

## Final Flag

`Kaal{7h15_c0d3_15_4n_4b50lu73_m355_4nd_1_5t1ll_d0n7_kn0w_h0w_17_w0rk5}`
