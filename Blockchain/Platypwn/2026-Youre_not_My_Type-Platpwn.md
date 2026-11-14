# You’re Not My Type — Writeup

## Bug

The payment backend converts the requested amount to a float, then checks the sender’s balance against `int(value)`:

```python
if balance < int(value):
    return None
```

For a request of `"0.999"`, `int(value)` is `0`. That means a sender with a zero balance passes the check. The backend then converts the full amount to wei and calls `settle`, which credits the recipient `0.999 ETH` and leaves the sender with a negative ledger balance.

Each sender can exploit this once. Use several fresh sender accounts and send to one recipient to accumulate enough balance to withdraw more than 20 ETH.

## Steps

1. Launch an instance and connect to its RPC endpoint.
2. Create a recipient account and fund it with enough ETH for gas.
3. Create fresh sender accounts. Fund each with a small amount of ETH for gas; they do not need a PlatyPay deposit.
4. From each sender, request a transfer of `"0.999"` ETH to the recipient.
5. Wait for the backend to process each request. Repeat until the recipient’s PlatyPay balance exceeds 20 ETH.
6. Have the recipient withdraw at least 20 ETH. The setup is solved when the target contract’s balance falls below 80 ETH.
7. Use the spawner’s **get flag** option with the instance UUID.

## Flag

`PP{n0t_my_typ3_0f_m0n3y::AA9vbjNWIj_H}`

---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
