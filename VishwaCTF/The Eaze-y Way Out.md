# The Eaze-y Way Out

## Challenge

- Name: The Eaze-y Way Out
- Type: Crypto
- Ciphertext: `AriozzYCQ{Vawr_Xjxggmpezlox}`
- Expected flag format: `VishwaCTF{}`

## Observations

The description gives two direct clues:

1. "look at it through a mirror" strongly suggests **Atbash**.
2. "the secret word in the challenge name" gives the key **`eaze`**.

## Solve Path

First apply Atbash to the ciphertext:

```text
AriozzYCQ{Vawr_Xjxggmpezlox}
ZirlaaBXJ{Ezdi_Cqcttnkvaolc}
```

That intermediate text still does not match the flag format, so the next step is to decrypt it with a **Vigenere cipher** using the key `eaze`.

Decrypting:

```text
Vigenere-decrypt(Atbash(ciphertext), key="eaze")
= VishwaCTF{Eaze_Cryptography}
```

## Verification

Re-encrypting the recovered plaintext with:

1. Vigenere encrypt using key `eaze`
2. Atbash

produces the original ciphertext exactly.

## Flag

`VishwaCTF{Eaze_Cryptography}`
