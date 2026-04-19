# CTF Writeup: One Time Pad
**Category:** Cryptography  
**Flag:** `HiveCTF{x0r_is_n0t_4_p4d_if_reus3d_pls_use_AES}`

---

## Challenge Description

> Our agent intercepted two encrypted transmissions. We know they use a "One-Time Pad," but we suspect the operator got lazy and reused the same key for both. Can you recover the messages?
>
> **Ciphertext 1:** `3757ec4497e41ebd990f00f7241a5ed26e5cac9448ec7f01b7447c2c841a4be912b0ef28aa926007db283640f90d55`  
> **Ciphertext 2:** `345bff51f4c937b3931f19cd341a21cf3b4b81c563bc2a0b8c0d6912901a1f`  
> **Hint:** Keep your keys secret and safe!

---

## Background

A **One-Time Pad (OTP)** is theoretically unbreakable — but only when used correctly. The rules are:

1. The key must be **truly random**
2. The key must be **at least as long as the message**
3. The key must **never be reused**

Breaking any one of these rules destroys the security. In this challenge, the operator broke **all three**.

Encryption is simply XOR: `C = P ⊕ K`

---

## Reconnaissance

First, note the ciphertext lengths:

```
C1: 47 bytes
C2: 31 bytes
```

The hint phrase `Keep your keys secret and safe!` is exactly **31 characters** — matching C2's length perfectly. This is a strong signal that C2 encrypts the hint phrase, and C1 (47 bytes) encrypts the flag.

---

## Step 1 — Two-Time Pad Attack (Key Reuse)

When the same key `K` encrypts two plaintexts:

```
C1 = P1 ⊕ K
C2 = P2 ⊕ K
```

XOR-ing the ciphertexts cancels the key entirely:

```
C1 ⊕ C2 = P1 ⊕ P2
```

This is the classic **"two-time pad"** vulnerability. The attacker now has the XOR of both plaintexts with no key material needed.

---

## Step 2 — Crib Dragging

With `P1 ⊕ P2` in hand, we can slide known plaintext "cribs" across it. If we guess a portion of one plaintext correctly, XOR-ing it against the corresponding position of `P1 ⊕ P2` reveals the other plaintext at that position.

Trying the crib `HiveCTF` at offset 0:

```
(P1 ⊕ P2)[0:7] ⊕ "HiveCTF" = "Keep yo..."
```

This matches the hint! Conversely, trying `Keep` at offset 0:

```
(P1 ⊕ P2)[0:4] ⊕ "Keep" = "Hive"
```

Both directions confirm: **C2 = encrypt("Keep your keys secret and safe!")** and **C1 = encrypt(flag)**.

---

## Step 3 — Key Recovery

Using the known plaintext P2 against C2 reveals the key stream:

```python
c2 = bytes.fromhex("345bff51f4c937b3931f19cd341a21cf3b4b81c563bc2a0b8c0d6912901a1f")
p2 = b"Keep your keys secret and safe!"
key = bytes(a ^ b for a, b in zip(c2, p2))
# key = 7f3e9a21d4b058c6e13f72a84d6901bc5e28f3a0179c4b65e82d1a73f67f3e
```

Decrypting the first 31 bytes of C1:

```python
p1_partial = bytes(a ^ b for a, b in zip(c1[:31], key))
# b'HiveCTF{x0r_is_n0t_4_p4d_if_reu'
```

The flag starts to emerge — but C1 is 47 bytes and our key is only 31. We need 16 more key bytes.

---

## Step 4 — Exploiting a Weak, Repeating Key

Examining the recovered key bytes closely:

```
Index:  0   1   2   3  ...  27  28  29  30
Key:   7f  3e  9a  21  ...  1a  73  f6  7f  3e
                                         ↑   ↑
                              Same as index 0 and 1!
```

Bytes 29 and 30 match bytes 0 and 1. The key has a **period of 29** — this is not a true OTP at all, but a repeating-key XOR cipher (essentially Vigenère). This is the third operator mistake.

Extending the key with period 29:

```python
period = 29
extended_key = bytes(key[i % period] for i in range(len(c1)))
```

---

## Step 5 — Full Decryption

```python
p1_full = bytes(a ^ b for a, b in zip(c1, extended_key))
# b'HiveCTF{x0r_is_n0t_4_p4d_if_reus3d_pls_use_AES}'
```

---

## Full Solve Script

```python
c1 = bytes.fromhex("3757ec4497e41ebd990f00f7241a5ed26e5cac9448ec7f01b7447c2c841a4be912b0ef28aa926007db283640f90d55")
c2 = bytes.fromhex("345bff51f4c937b3931f19cd341a21cf3b4b81c563bc2a0b8c0d6912901a1f")

# Step 1: Use known plaintext (hint) to recover key
p2 = b"Keep your keys secret and safe!"
key = bytes(a ^ b for a, b in zip(c2, p2))

# Step 2: Detect key period (period = 29)
period = 29
assert all(key[i] == key[i % period] for i in range(len(key)))

# Step 3: Extend key and decrypt flag
extended_key = bytes(key[i % period] for i in range(len(c1)))
flag = bytes(a ^ b for a, b in zip(c1, extended_key))

print(f"C2 plaintext : {p2.decode()}")
print(f"C1 plaintext : {flag.decode()}")
```

**Output:**
```
C2 plaintext : Keep your keys secret and safe!
C1 plaintext : HiveCTF{x0r_is_n0t_4_p4d_if_reus3d_pls_use_AES}
```

---

## Summary of Operator Mistakes

| # | Mistake | Why It's Fatal |
|---|---------|---------------|
| 1 | **Reused the key** for two messages | Enables two-time pad attack; key cancels out via XOR |
| 2 | **Short key (29 bytes)** shorter than the 47-byte message | Key wraps around, turning OTP into a weak repeating-key cipher |
| 3 | **Used predictable plaintext** (the hint itself) as one of the messages | Directly exposed the key via known-plaintext attack |

---

## Flag

```
HiveCTF{x0r_is_n0t_4_p4d_if_reus3d_pls_use_AES}
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
