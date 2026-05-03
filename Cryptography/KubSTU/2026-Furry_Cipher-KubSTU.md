# Furry Cipher Writeup

## Challenge Info

- **Name:** Furry Cipher
- **Category:** Crypto
- **Points:** 988
- **Flag format:** `KubSTU(...)`

## Summary

The archive contains a small Python encryption script and a very large text file. The core trick is that the huge text file is mostly decoy noise. The real ciphertext is hidden inside it as rare alphanumeric characters plus the flag separators `(`, `)` and `_`.

Once those rare characters are extracted in order, the result is a short ciphertext:

```text
XiEDJ5(9tV_qY3_v43_t9B3_o9vo_ESM_YR_YA_t_S5t8v_XYL4jt)
```

Using the known flag prefix `KubSTU(` and the provided encryption routine, we can recover the 3-value key and decrypt the full string.

## Provided Files

The zip archive contains:

- `Furry Cipher.py`
- `Weird_Furry_text.txt`

The Python file defines a custom substitution over the alphabet:

```python
alphabet = string.ascii_uppercase + string.ascii_lowercase + string.digits
```

Encryption depends on the character index modulo `3`:

```python
if i % 3 == 0:
    encrypted = (num * 13 + key_val * 7) % 62
elif i % 3 == 1:
    encrypted = (num * 17 + key_val * 3 + 11) % 62
else:
    encrypted = (num * 19 + (key_val ^ 42) + 23) % 62
```

Characters in `()_` are preserved.

## Key Observation

`Weird_Furry_text.txt` is about `1.13 GB` uncompressed and looks like meaningless punctuation. Sampling shows only this small symbol alphabet appears in the bulk of the file:

```text
! " # & * + , . / < = > ? @ \ ^ | №
```

That is inconsistent with the provided Python cipher, because the Python routine only outputs characters from:

- `A-Z`
- `a-z`
- `0-9`
- `(`, `)`, `_`

So the giant punctuation stream cannot be the direct output of the script. That suggests the actual ciphertext is embedded sparsely inside the noise.

## Extracting the Real Ciphertext

Scanning the entire text stream for only `[A-Za-z0-9()_]` reveals that such characters are extremely rare.

The extracted sequence is:

```text
XiEDJ5(9tV_qY3_v43_t9B3_o9vo_ESM_YR_YA_t_S5t8v_XYL4jt)
```

This already matches the expected flag shape.

## Recovering the Key

We know the plaintext starts with `KubSTU(`. Comparing this known prefix against the extracted ciphertext prefix `XiEDJ5(` lets us recover the 3 key values used by the cyclic scheme.

Recovered key:

```text
[29, 57, 48]
```

## Decryption

After inverting each affine transformation by position modulo `3`, the ciphertext decrypts to:

```text
KubSTU(h0w_d1d_you_re4d_7ha7_br0_1t_1s_a_furry_c1pher)
```

## Flag

```text
KubSTU(h0w_d1d_you_re4d_7ha7_br0_1t_1s_a_furry_c1pher)
```

## Minimal Solver

```python
import io
import re
import string
import zipfile

alphabet = string.ascii_uppercase + string.ascii_lowercase + string.digits
char_map = {ch: i for i, ch in enumerate(alphabet)}
num_map = {i: ch for i, ch in enumerate(alphabet)}

zip_path = "Furry_Cipher.zip"
member = "Furry Cipher/Weird_Furry_text.txt"

with zipfile.ZipFile(zip_path) as zf:
    with zf.open(member) as raw:
        data = io.TextIOWrapper(raw, encoding="utf-8").read()

ciphertext = "".join(re.findall(r"[A-Za-z0-9()_]", data))
print("ciphertext:", ciphertext)

known = "KubSTU("
key = [None, None, None]

for i, (p, c) in enumerate(zip(known, ciphertext)):
    if p in "()_" or c in "()_":
        continue
    pn = char_map[p]
    cn = char_map[c]
    if i % 3 == 0:
        for kv in range(62):
            if (pn * 13 + kv * 7) % 62 == cn:
                key[0] = kv
                break
    elif i % 3 == 1:
        for kv in range(62):
            if (pn * 17 + kv * 3 + 11) % 62 == cn:
                key[1] = kv
                break
    else:
        for kv in range(62):
            if (pn * 19 + (kv ^ 42) + 23) % 62 == cn:
                key[2] = kv
                break

print("key:", key)

inverse = [{}, {}, {}]
for pn in range(62):
    inverse[0][(pn * 13 + key[0] * 7) % 62] = pn
    inverse[1][(pn * 17 + key[1] * 3 + 11) % 62] = pn
    inverse[2][(pn * 19 + (key[2] ^ 42) + 23) % 62] = pn

plaintext = []
for i, ch in enumerate(ciphertext):
    if ch in "()_":
        plaintext.append(ch)
    else:
        cn = char_map[ch]
        pn = inverse[i % 3][cn]
        plaintext.append(num_map[pn])

print("flag:", "".join(plaintext))
```

## Takeaway

This was not a difficult cipher mathematically. The real obstacle was recognizing that the bulk text was a distraction layer and that the actual encrypted message was hidden as sparse outlier characters inside the file.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
