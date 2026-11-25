## Writeup

**The setup.** `DeCipher` is AES-128 with a custom S-box. `output_stage2.npz` has 16 plaintext/ciphertext pairs, and the flag is AES-GCM encrypted under `sha256(all 11 round keys)`. The "execution proofs" are the part that leaks. For each pair of encryptions *(i, j)*, it contains the XOR of the two cipher states at every round. In stage 2 the key is no longer wrapped, so recovering the round keys is the only way to the flag.

**The weakness is the S-box.** It is a valid permutation, but it is almost linear:
- 63 nonzero shifts `a` have `S(x) ⊕ S(x⊕a)` constant for every `x`. Together with 0 they form a 6-dimensional subspace `L`.
- Every byte can therefore be split as `x = rep ⊕ l`, with `rep` one of `{0, 2, 64, 66}` and `l` in `L`.
- That gives `S(x) = M(x) ⊕ F(q(x))`. `M` is an 8×8 linear map over GF(2), `q(x)` is a 2-bit linear function, and `F` is just a 4-entry table.

So the only nonlinear part of each S-box is 2 bits.

**Why the proofs don't hand over the key immediately.** The difference `S(a⊕k) ⊕ S(b⊕k)` depends on `k` only through `q(k)`, the 2 bits identifying its coset of `L`. Round 0 narrows each key byte to 64 candidates, not 1. A direct round-by-round attack therefore stalls, and I also tried z3 on the full cipher, which didn't converge in time.

**The solution.**
1. For each round and each state byte, test the 4 possible values of `q` against the proof differences of all 15 other encryptions. Exactly one value survives every time (160 bytes in total), so every S-box in the cipher body becomes `M(x)` plus a known constant.
2. With those `q` values known, the whole cipher is affine over GF(2) in the base key bits.
3. The key schedule also calls the S-box, 40 times. For each of those calls I added three variables `A=qa`, `B=qb`, `P=A·B`. They are tied to the key by linear equations, and I left `P` unconstrained so the system stays linear.
4. The linear equations are:
   - 320 equations: the known `q` bits of every cipher S-box input (2 bits × 16 bytes × 10 rounds)
   - 128 equations: the final `ciphertext = state ⊕ rk10`
   - 80 equations: tying `A` and `B` to the key schedule
5. Gaussian elimination over GF(2) on 248 unknowns (128 key bits + 120 auxiliary) gives rank 248. The solution is unique, and `P` comes out consistent without being forced.

**Final step.** Run the real key schedule on the recovered key, hash the 176 bytes of round keys with SHA-256, and decrypt the flag with AES-GCM using a zero nonce.

The flag text ("gaus5 and g41oi5 to the rescue") matches the intended solution: Gauss for the linear algebra and Galois for the field structure.

The solver is at `/home/claude/solve2g.py`. I can package it with this writeup if you want.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
