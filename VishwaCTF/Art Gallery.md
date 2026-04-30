# Art Gallery Writeup

## Challenge

- Name: `Art Gallery`
- Type: Android reverse engineering

## Recon

The APK exposes three activities in the manifest:

- `MainActivity`
- `GalleryActivity`
- `CanvasActivity`

`GalleryActivity` sets a long-press handler on the displayed image. Long-pressing launches `CanvasActivity`, which is the restricted metadata panel mentioned in the prompt.

## Java Layer

`CanvasActivity` does three relevant things:

1. Loads `libartvault.so`
2. Computes a `runtimeSignal`
3. Calls two native methods:

```java
private native boolean verifyGateNative(int i);
private native byte[] decryptFlagNative(int i);
```

The Java code returns `ACCESS_DENIED` unless:

- no debugger is attached
- `verifyGateNative(runtimeSignal)` is true
- `decryptFlagNative(runtimeSignal)` returns non-empty bytes

## Native Gate

`verifyGateNative` is very small. It applies a Murmur-style finalizer to:

```text
runtimeSignal ^ 0x6c8e9cf5
```

and checks the result against:

```text
0xd15ea5ed
```

Because the finalizer is invertible modulo `2^32`, the expected `runtimeSignal` can be recovered offline:

```text
runtime_signal = 0x6c691766
```

## Native Decryption

`decryptFlagNative` does not actually use the `runtimeSignal` value after the gate. The flag is recovered entirely from static tables in `libartvault.so`.

The process is:

1. Build a 33-byte lookup table by interleaving three 11-byte blobs from `.rodata`
2. Use a 33-byte permutation table to reorder that lookup table
3. For each byte:
   - advance an LCG state
   - XOR with a per-position mask
   - substitute both nibbles through a 16-byte S-box
   - rotate right by `(index % 5) + 1`

Re-implementing that logic yields the plaintext directly.

## Solve Script

`solve.py` in this directory reproduces the full offline recovery.

Run:

```bash
python3 solve.py
```

## Flag

```text
VishwaCTF{secret_gallery_exposed}
```
