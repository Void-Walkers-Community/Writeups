# Just in Time — Writeup

The vault’s password is generated when `Chal` is deployed:

```solidity
uint256 password = uint256(
    keccak256(abi.encodePacked(block.timestamp, block.number, msg.sender))
);
passwordHash = keccak256(abi.encodePacked(password));
```

The password is not stored in plaintext, but its inputs are available from the chain. `Chal` is created inside `Setup`’s constructor, so the `msg.sender` used by `Chal` is the **Setup contract address**.

## Recover the password

From the deployment transaction and block, collect:

- Deployment block number: `2`
- Block timestamp: `1794628667`
- `msg.sender`: the Setup contract address

Reproduce Solidity’s packed encoding: timestamp as 32 bytes, block number as 32 bytes, and the address as 20 bytes. Hash those bytes with Ethereum Keccak-256 and interpret the result as an integer.

This gives the password:

```text
64449600685036012754367370378437671482182243963659445467576610331204069857007
```

## Withdraw and verify

Call `withdraw(password)` from the challenge account. The transaction succeeded, the vault balance became zero, and `Setup.isSolved()` returned `true`.

## Flag

```text
PP{t1m3_15_n0t_4_s3cr3t::iYhHAxYTEmCx}
```
---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

