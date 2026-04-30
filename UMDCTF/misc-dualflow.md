# misc/dualflow writeup

## Flag

`UMDCTF{a_little_gram_schmidt_never_hurt_anybody}`

## Recon

The handout gives:

- `challenge.py`
- `flow.py`
- `flow_0.pt`
- `flow_1.pt`
- `reference_window.npy`
- `calibration.npz`

The checker is:

1. Input must have shape `(5, 64)` and stay within `L_inf <= 0.08` of `reference_window.npy`.
2. Let `q_0` and `q_1` be the two RealNVP log densities.
3. Accept iff:
   - `q_1 - q_0 >= 30`
   - `q_1 >= lp_thresh1`
   - `log|det J_f1|` stays inside the calibration band

So this is a constrained optimization problem over a very small box around the reference sample.

## Main observation

At the reference point, `q_1` is already comfortably above threshold and `log|det J_f1|` is already inside band. The only thing that fails is the margin.

Directly maximizing `q_1 - q_0` is a trap. The margin gradient mostly points in a direction that destroys `q_1` and falls off the realistic manifold almost immediately.

What we actually want is:

- push `q_0` down
- keep the first-order change to `q_1` near zero

That is exactly a Gram-Schmidt style projection.

## Exploit

I decoded the `.pt` checkpoints without PyTorch by reading the zip/pickle format directly and reimplemented the RealNVP forward pass plus exact reverse-mode gradients in NumPy.

Then:

1. Compute gradients at the reference point:
   - `g0 = grad q_0`
   - `g1 = grad q_1`
2. Start from the descent direction for `q_0`, which is `-g0`.
3. Remove its component along `g1`:

```text
v = -g0 - proj_{g1}(-g0)
  = -g0 - ((-g0 · g1) / (g1 · g1)) g1
```

This gives a direction that mostly lowers `q_0` while keeping the first-order change in `q_1` close to zero.

After that, scan a tiny scalar `alpha` along this 1D direction:

```text
x = clip(x_ref + alpha * v_normalized, x_ref - 0.08, x_ref + 0.08)
```

There is a narrow valid interval around `alpha ~= 3.6e-4`.

My final local candidate had:

- `margin = 31.845648`
- `q1 = 947.589966`
- `log_det1 = 1518.546918`
- `linf = 0.000362`

which satisfied the remote checker directly.

## Files

- Solver: [solve.py](/home/kali/ctf-orc/ctf/misc-dualflow/solve.py)
- Writeup: [writeup.md](/home/kali/ctf-orc/ctf/misc-dualflow/writeup.md)

## Remote result

Running:

```bash
python solve.py --remote --host challs.umdctf.io --port 30303
```

returned:

```text
FLAG: UMDCTF{a_little_gram_schmidt_never_hurt_anybody}
```
