# AE-no-S: Why the "S" Actually Matters

The challenge title jokes that the **S in AES stands for Standard, not SubBytes**, implying that SubBytes might be unnecessary.

Spoiler: it absolutely is necessary.

## Initial Analysis

Looking at the provided source code, two important modifications were made to AES:

1. **SubBytes was completely removed** from every round.
2. **SubWord was removed** from the key schedule.

In the encryption routine:

```python
for round_index in range(1, N_ROUNDS):
    _shift_rows(state)
    _mix_columns(state)
    _add_material(state, schedule[round_index])
```

Normally AES would perform:

```python
SubBytes
ShiftRows
MixColumns
AddRoundKey
```

but here SubBytes is replaced with the identity function.

The key schedule is similarly weakened:

```python
if index % 4 == 0:
    temp = _rot_word(temp)
    temp[0] ^= RCON[index // 4 - 1]
```

Normally AES would also apply SubWord here.

---

## Why This Breaks Security

The only nonlinear component of AES is the S-box.

Without SubBytes and SubWord:

* ShiftRows is linear.
* MixColumns is linear.
* XOR with round keys is linear.
* Key expansion becomes linear.

Therefore the entire cipher becomes an affine transformation:

[
E(P)=A\cdot P \oplus B
]

where:

* (P) is the plaintext vector
* (A) is a 128×128 binary matrix
* (B) is a constant vector

This means encryption behaves like one giant linear function.

---

## Using the Given Data

The challenge provides:

```json
"zero": {
    "pt": "0000...",
    "ct": "b884..."
}
```

and 128 basis vectors:

```json
{
    "pt": "80000000...",
    "ct": "a384..."
}
```

Each basis plaintext contains exactly one bit set.

Because the cipher is affine:

[
E(e_i)\oplus E(0)
]

gives the (i)-th column of the matrix (A).

So:

1. Take the ciphertext of the all-zero block.
2. XOR every basis ciphertext with it.
3. Recover all 128 columns of (A).

After processing all basis pairs, the complete encryption matrix is known.

---

## Building the Inverse

Once (A) is known, compute:

[
A^{-1}
]

using Gaussian elimination over GF(2).

For any ciphertext:

[
P=A^{-1}(C\oplus B)
]

where:

[
B=E(0)
]

This immediately gives the plaintext.

---

## Decrypting the Flag

The provided encrypted flag was:

```text
cfbd18fe758f3a7a9c5f996aeec952b049f49297cf364b8542457403cc7be777c8778ae5adfabcf13edf844fac7b27c7
```

Splitting into 16-byte blocks and applying:

[
P=A^{-1}(C\oplus B)
]

to each block recovers the original message.

The recovered key is:

```text
47c8cfbcf51a6952fd4204f762e3c9d1
```

and decrypting the ciphertext reveals the flag.

---

## Takeaway

AES relies heavily on the S-box for security.

Removing SubBytes doesn't make AES "slightly weaker"—it destroys the nonlinearity that prevents linear algebra attacks. The cipher collapses into a simple affine transformation that can be fully recovered using only encryptions of the zero block and the 128 basis vectors.

In other words:

> The S in AES may stand for "Standard", but the S-box is the reason AES is secure. 😄

---
* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
