# ECB-Lasagna Writeup (bctf)

## Challenge intuition

The name **ecb-lasagna** is a clue:

* **ECB** → likely AES-ECB involved.
* **Lasagna** → multiple layers.

The challenge is about peeling back layers of transformations.

---

## Files

The archive contains:

* `chall.py`
* `output.txt`
* `flag.txt`

### Decoy flag

`flag.txt` contains:

```text
bctf{fake_flag}
```

This is intentionally fake.

In `chall.py`, the real flag is loaded from:

```python
open("../flag.txt")
```

That means the actual flag is outside the distributed files.

So the only route is reversing the encryption logic.

---

## Step 1 — Read the encryption

The challenge repeatedly applies:

1. XOR transformations
2. AES-ECB encryption
3. Additional XOR mixing
4. Repeated layering ("lasagna")

Conceptually:

```text
flag
 ↓
XOR layer
 ↓
AES-ECB
 ↓
XOR layer
 ↓
AES-ECB
 ↓
... repeated ...
 ↓
output.txt
```

---

## Step 2 — Why ECB matters

ECB encrypts blocks independently:

```text
C_i = AES(K, P_i)
```

To reverse:

```text
P_i = AES⁻¹(K, C_i)
```

So every AES layer can be peeled off block-by-block.

---

## Step 3 — Reverse order matters

Encryption did:

```text
XOR
AES
XOR
AES
```

So decryption must do the exact reverse:

```text
AES-decrypt
undo XOR
AES-decrypt
undo XOR
```

Last layer added = first layer removed.

Classic layered crypto reversal.

---

## Step 4 — Undo the XOR layers

For each XOR stage:

```python
plaintext = ciphertext ^ key
```

Because:

```text
A ^ K ^ K = A
```

XOR undoes itself.

This removes each mixing layer.

---

## Step 5 — Peel all lasagna layers

Repeat:

```python
while layers_remaining:
    data = aes_ecb_decrypt(data)
    data = undo_xor(data)
```

Eventually the recovered plaintext starts with:

```text
bctf{
```

which confirms success.

---

## Recovered flag

```text
bctf{y0u'v3_h349d_0f_5p4gh3tt1_c0d3,_8ut_d1d_y0u_kn0w_l454gn4_c0d3_4150_3x15t5?_1t_m34n5_y0u_c4n't_m4k3_4_ch4ng3_50m3wh3r3_w1th0ut_m4k1n6_4_ch4ng3_1n_m4n7_0th3r_p4rt5_0f_th3_c0d5.}
```

---

## Solver outline (simplified)

```python
from Crypto.Cipher import AES

ct = open("output.txt","rb").read()

for layer in reversed(layers):
    if layer.type == "aes":
        cipher = AES.new(layer.key, AES.MODE_ECB)
        ct = cipher.decrypt(ct)

    elif layer.type == "xor":
        ct = bytes(a^b for a,b in zip(ct, layer.key_stream))

print(ct)
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
