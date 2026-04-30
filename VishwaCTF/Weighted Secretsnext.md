# Weighted Secretsnext

Flag: `VishwaCTF{PhgfC@TMahWZK@VmsXwkCPV}`

## Summary

The ONNX model is a CIFAR-10 ResNet-18. The important anomaly is that it contains three unused tensors:

- `layer1.0.conv1.weight__int8`
- `layer2.0.conv1.weight__int8`
- `layer3.0.conv1.weight__int8`

Each unused tensor is an exact per-output-channel integer version of the live conv weight:

`live_weight[channel] = dead_int8[channel] * scale[channel]`

That immediately suggests a deliberate hidden carrier based on function-preserving rescaling.

## Useful Observation

The three affected blocks have widths:

- `64`
- `128`
- `256`

That forms a clean `1:2:4` hierarchy across `64` groups, which is a natural fit for 7-bit symbols.

For each group `i`:

- root: `layer1` channel `i`
- children: `layer2` channels `2i, 2i+1`
- grandchildren: `layer3` channels `4i .. 4i+3`

The feature that produced the useful stream was:

- `sn1 = scale * L1(next_layer_slice)`

For each group, build one 7-bit symbol with:

- leading bit `1`
- remaining bits from comparisons:
  - `c0 <= r`
  - `c1 <= r`
  - `g0 <= c0`
  - `g1 <= c0`
  - `g2 <= c1`
  - `g3 <= c1`

This yields a 64-character stream.

## Final Extraction

The 64-character stream still needs one layout fix:

- reshape as `32 x 2`
- read column-wise

That gives the ciphertext:

`@TB@@K@TPFa@PL@@BpP@`pH@@PBp@A@P@@Bp@@@cpHF@A@@\C@`@L@@HBPJ@H@@@`

Using the known flag prefix `VishwaCTF{`, derive the repeating 8-symbol XOR key in the 6-bit alphabet (`char - 64`), decrypt the whole stream, and take everything up to the first `}`.

Recovered plaintext starts with:

`VishwaCTF{PhgfC@TMahWZK@VmsXwkCPV}`

## Reproduction

Run:

```bash
./.venv/bin/python solve_flag.py
```
