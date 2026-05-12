Exponope 
The RSA implementation used a very small public exponent and no padding.
Because the plaintext was smaller than (N^{1/e}), the encryption effectively became:

c = m^e

So the ciphertext was just the plaintext raised to the power of `e` (likely `e = 5`) without modular reduction happening.

That means we can simply recover the message by taking the integer 5th root of the ciphertext.

Recovered flag:

`THC{u_n3eD_@_bett3r_eXp0neNT}`

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
