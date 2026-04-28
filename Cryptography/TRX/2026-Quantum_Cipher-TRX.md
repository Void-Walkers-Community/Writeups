# Quantum Cipher — CTF Writeup

**Category:** Crypto  
**Author:** v3r7ux  
**Flag:** `TRX{1NV4R14N7_5U85P4C35_4ND_UN174R17Y}`

---

## What is the challenge?

We get a Python server. It uses a **quantum-inspired** encryption. We can:

1. Encrypt **one byte** of our choice (only 1 query allowed!)
2. Get the **encrypted flag**

The flag is encrypted block by block (8 bits at a time). We need to find all the secret bytes to decrypt it.

---

## How does the cipher work?

The cipher uses 16 qubits total:
- **8 data qubits** — holds the plaintext byte
- **8 key qubits** — holds a secret byte from `self.secret`

Each encryption runs **12 rounds**. Every round does three things:

1. **add_key** — applies a small rotation (`CRY` gate, angle `π/15`) to each data qubit, controlled by a key qubit
2. **apply_permutation** — shuffles the data qubits using `iswap` gates in a fixed pattern
3. **apply_rxz_interaction** — applies `RZX` rotations between fixed qubit pairs

The key evolves each round: `secret[0]` → `secret[1]` → ... → `secret[12]`.

After 12 rounds, the server takes **256 amplitudes** from a fixed position in the 65536-element statevector and returns them as the ciphertext.

---

## The Key Insight

The key qubits **never leave basis states**. A `CRY` gate does nothing if the control qubit is `|0⟩`. If the key bit is `0`, the rotation is skipped. If it is `1`, the rotation is applied.

This means:
- The full transformation is a **fixed unitary matrix U** that depends only on the 96 secret bits (`secret[0]` through `secret[11]`)
- `secret[12]` only controls the output index, but we can figure it out from the data

Each output block is `U|b⟩` — the unitary applied to a basis state `|b⟩`.

---

## The Attack — Known-Plaintext + Greedy Search

### Step 1 — Collect data

We have **1 free query** + we know the flag starts with `TRX{`.

- Encrypt `0x00` → gives us `U|0⟩` (the first column of U)
- Get encrypted flag → gives us `U|T⟩`, `U|R⟩`, `U|X⟩`, `U|{⟩`, and all the unknown bytes

That gives us **5 known pairs**: `(plaintext, ciphertext)`.

### Step 2 — Simulate the cipher locally

Since Qiskit was not available locally, a **numpy simulator** was written from scratch. It replicates:
- `CRY` gate (controlled-RY)
- `iswap` gate
- `RZX` gate

The qubit axis mapping (`axis = 7 - qubit`) matches Qiskit's statevector convention.

### Step 3 — Recover the secret with greedy bit-flipping

We define a distance function:

```
dist(secret) = Σ ||simulate(secret, b) - ct[b]||²
```

We start with `secret = [0, 0, 0, ..., 0]` (all zeros) and flip one bit at a time. If flipping a bit reduces the distance, we keep the flip and repeat.

This greedy search converges quickly because the rotation angle `π/15` is small — the optimization landscape is smooth enough for greedy search to work.

After a few dozen flips, the distance drops to nearly `0`.

### Step 4 — Decrypt the flag

With the secret recovered, we simulate `U|b⟩` for all 256 possible bytes. Then for each block of the encrypted flag, we find the byte `b` whose simulation matches closest:

```python
for i in range(len(ct_flag) // 256):
    target = ct_flag[i*256 : (i+1)*256]
    best_b = min(range(256), key=lambda b: ||all_states[b] - target||²)
    flag += chr(best_b)
```

---

## The Solve Script (key parts)

```python
import numpy as np

# Simulate one block: apply 12 rounds to a starting byte
def simulate(secret_bytes, initial_byte):
    state = np.zeros(256, dtype=complex)
    state[initial_byte] = 1.0
    state = state.reshape([2]*8)
    
    for r in range(12):
        s = secret_bytes[r]
        # add_key: apply small rotation if key bit is 1
        for i in range(8):
            if (s >> KEY1_INDEX[i]) & 1:
                state = apply_gate_1q(state, get_ry(np.pi/15), i)
        # permutation + rxz interaction
        for i in range(8):
            state = apply_gate_2q(state, get_iswap(), PERMUTATION[i], PERMUTATION[(i+1)%8])
        for c, t in RXZ_INTERACTION:
            state = apply_gate_2q(state, get_rzx(np.pi/np.e), c, t)
    
    return state.flatten()

# Greedy bit-flip to recover secret
current_s = [0]*12
for round in range(12):
    for bit in range(8):
        new_s[round] ^= (1 << bit)
        if dist(new_s) < dist(current_s):
            current_s = new_s  # keep the flip
```

---

## Why does greedy search work?

The `CRY` gate uses a very small angle (`π/15 ≈ 0.21 radians`). Each key bit only slightly rotates the state. This means:

- Wrong bits cause small, independent errors in the output
- The error from one wrong bit does not hide the effect of another wrong bit
- Flipping bits one by one can find the correct secret without getting stuck

If the rotation angle were large (e.g., `π/2`), greedy search would likely fail.

---

## How to fix it

- **Use a large rotation angle** — small angles make each bit's effect independently visible and easy to recover
- **Mix key bits non-linearly** — if each round depends on all previous key bits in a complex way, a simple greedy search cannot recover them bit by bit
- **Use a proper key schedule** — the secret key should not be 12 independent bytes; it should be derived from a single master key with a cryptographic key derivation function

---


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
