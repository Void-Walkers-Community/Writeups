# PlatyLocker writeup

The contract claims it will decrypt the AES key after a 1000 ETH payment. But the RSA private exponent is stored on the blockchain. Solidity’s `private` keyword only restricts access through the contract interface; it does not hide storage from `eth_getStorageAt`.

## Recover the AES key

The `Chal` storage layout is:

| Slot | Value |
|---|---|
| 0 | RSA modulus `n` (`bytes`, long form) |
| 1 | Public exponent `e` (`bytes`, inline short form) |
| 2 | RSA private exponent `d` (`bytes`, long form) |
| 3 | Encrypted AES key `c` (`bytes`, long form) |
| 4 | `keyHash` |

For long `bytes` values, the slot stores `length * 2 + 1`; the data starts at `keccak256(slot)`. The public exponent is only three bytes, so Solidity stores it directly in slot 1, left-aligned.

Then decrypt using the same RSA operation as the contract:

`key = c^d mod n`

## Restore the files

Call `restoreFiles(key)` with the recovered 32-byte key. This checks `keccak256(key)` against the on-chain `keyHash`, so you can verify the decryption before submitting the transaction. No ransom payment is required.

After restoration, choose **3 — get flag** in the challenge spawner.

**Flag:** `PP{pr1v4t3_15_n0t_s3cr3t::Um8Yby7WDIZG}`

---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
