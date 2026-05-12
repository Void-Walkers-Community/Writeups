The encryption used a weird “min-plus matrix” operation instead of normal matrix multiplication.

For each ciphertext value:

c_i = \min_j (K_{ij} + m_j)

The server leaked both the key matrix `K` and the ciphertext `ct` through the `status` option, so the cipher became reversible.

From the equation:

m_j \ge c_i - K_{ij}

Taking the maximum over all rows gives the exact plaintext byte:

m_j = \max_i (c_i - K_{ij})

So for every ciphertext block, we reconstructed each plaintext byte using:

```python id="w0t4tp"
m[j] = max(c[i] - K[i][j] for i in range(8))
```

After recovering all blocks and sending the resulting byte array back to the service, it returned the flag:

`THC{fl0yd_w4rsh4ll_m33ts_crypt0gr4phy_1n_th3_tr0p1cs}`

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
