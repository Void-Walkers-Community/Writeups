Unlucky 13 --> KubSTU{unLucky_13_l4y3r5_0f_encrypt10n_n0_luck_h3r3}
**KubSTU CTF — "Unlucky 13" Writeup**
When I first looked at the script, my eye went straight to `e = 3` in the RSA section. That's almost always a red flag in CTF crypto. Sure enough, when I compared the size of `c` to `n`, the ciphertext was suspiciously tiny — small enough that `m³` never even reached the modulus. So there was no modular reduction at all, meaning `c` was literally just `m³`. A plain integer cube root handed back the plaintext. No fancy lattice attacks, no factoring — just `iroot(c, 3)`.

With the RSA layer stripped off, I looked at the "forgotten_cipher." It's RC4, almost word for word. The name was a cheeky hint. RC4 is a stream cipher, so encryption and decryption are the same function — just run it again with the same key. The key itself was entirely deterministic: `SHA256(b"Unlucky13")[:16]`, where `13` was hardcoded in the source. Nothing to crack there.

The last layer was an XOR against a custom PRNG — an LCG seeded with, again, the constant `13`. XOR is its own inverse, PRNG seed is known, so reversing it was just re-running the same function.

The challenge was stacked to look intimidating — three layers, a custom cipher, RSA — but every single piece had a fatal flaw baked in. The seed was hardcoded, the cipher was a known algorithm in disguise, and the RSA parameters made a textbook mistake. Once you noticed that `c < n^(1/3)`, the rest unraveled quickly.

**Flag:** `KubSTU{unLucky_13_l4y3r5_0f_encrypt10n_n0_luck_h3r3}`

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
