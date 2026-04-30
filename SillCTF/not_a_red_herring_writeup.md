# Not a Red Herring

Type: Rev

## Summary

The challenge URL was a `shorturl.at` link. Querying the `www` host directly exposed the real target: a Penn State SharePoint folder share named `Not a Red Herring`.

That shared folder contained a single file:

```text
silly-1.20.1-forge-mod.jar
```

Decompiling the mod showed two relevant custom items:

- `ExampleMod$1` stored `SECRET_KEY = "The_Key"`
- `ExampleMod$2` stored an `encrypted_flag` integer array

The main class also contained a dormant XOR decryptor routine that made the intended solve obvious.

## Recon

Resolve the short link:

```bash
curl -sI https://www.shorturl.at/pZEPE
```

This returned a `Location` header pointing to a SharePoint share.

The share page exposed an anonymous drive API URL and scoped token in `_spPageContextInfo`, which allowed listing the shared folder and downloading the JAR from the terminal.

## Solve

Decompile or inspect the class files:

```bash
javap -classpath silly-1.20.1-forge-mod.jar -p -c 'com.example.examplemod.ExampleMod$1'
javap -classpath silly-1.20.1-forge-mod.jar -p -c 'com.example.examplemod.ExampleMod$2'
```

Relevant values:

```text
SECRET_KEY = "The_Key"
encrypted_flag = [39,1,9,51,50,6,13,50,19,43,48,31,58,24,11,58,86,59,20,45,28,38,26,68,49,44,24]
```

Apply the XOR routine from the mod:

```bash
python3 - <<'PY'
key='The_Key'
enc=[39,1,9,51,50,6,13,50,19,43,48,31,58,24,11,58,86,59,20,45,28,38,26,68,49,44,24]
print(''.join(chr(v ^ ord(key[i % len(key)])) for i, v in enumerate(enc)))
PY
```

Output:

```text
sillyctf{NoT_a_R3d_Herr!ng}
```

## Flag

```text
sillyctf{NoT_a_R3d_Herr!ng}
```
