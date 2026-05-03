nintendo 3ds --> KubSTU{3d3s_n1nt3nd0_cbc_m0d3_n07_h4rd_3n0ugh}
# CTF Writeup: 3DES CBC Decryption Challenge

## Overview

The challenge provided an encrypted blob along with several encoded values and two IVs. The goal was to identify the encryption algorithm, reconstruct the key, determine the correct IV, and decrypt the ciphertext.

---

## Step 1: Identifying the Algorithm

The file header stated `CBC+PKCS5`, indicating the mode and padding scheme. The flavor text hinted at something "very similar to Nintendo 3DS." This is a wordplay: **3DS** maps to **3DES** (Triple DES), an encryption algorithm that applies DES three times.

---

## Step 2: Decoding the Key

Three numbered values were provided, each encoded differently:

| Field | Raw Value | Encoding | Decoded |
|---|---|---|---|
| 1 | `TjFudDNuZG8=` | Base64 | `N1nt3ndo` |
| 2 | `83 51 99 117 114 49 116 121` | Decimal ASCII | `S3cur1ty` |
| 3 | `4b33792132303236` | Hex | `K3y!2026` |

Concatenated in order: `N1nt3ndoS3cur1tyK3y!2026` — exactly 24 bytes, a valid 3DES key.

---

## Step 3: Determining the IV

Two IVs were given:

- `ivx = 0a001f0273760054` (hex, 8 bytes)
- `ivm = M4r10Br0` (ASCII, 8 bytes — another Nintendo reference: Mario Bros)

Neither IV alone produced a valid plaintext for the first block. The correct approach was to XOR them together:

```
0a 00 1f 02 73 76 00 54  (ivx)
4d 34 72 31 30 42 72 30  (ivm as hex)
= 47 34 6d 33 43 34 72 64  => "G4m3C4rd"
```

The real IV is `G4m3C4rd`.

---

## Step 4: Decryption

With all parameters known:

- **Algorithm:** 3DES
- **Mode:** CBC
- **Padding:** PKCS5
- **Key:** `N1nt3ndoS3cur1tyK3y!2026`
- **IV:** `G4m3C4rd`

```python
from Crypto.Cipher import DES3

key = b"N1nt3ndoS3cur1tyK3y!2026"
iv  = b"G4m3C4rd"
ct  = bytes.fromhex("072a8e75459a545679f3aa56a9fafb38..."  )

cipher = DES3.new(key, DES3.MODE_CBC, iv)
pt = cipher.decrypt(ct)
pt = pt[:-pt[-1]]  # strip PKCS5 padding
print(pt)
```

---

## Result

```
KubSTU{3d3s_n1nt3nd0_cbc_m0d3_n07_h4rd_3n0ugh}
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
