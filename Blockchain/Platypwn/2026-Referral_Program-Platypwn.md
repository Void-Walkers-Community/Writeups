## Referral Program writeup

The goal is to drain the proxy’s 100 ETH. `Setup.isSolved()` returns true once the proxy balance is zero.

The proxy stores its `implementation` address in storage slot 0. The delegated `Chal` implementation also uses slot 0 for `latestReferrer`. This storage collision lets us change the proxy’s implementation through `deposit(referrer)`.

### Exploit

1. Deploy a tiny helper contract whose runtime executes `SELFDESTRUCT` to send the caller’s balance to your address.
2. Call `deposit(helperAddress)` on the proxy. Under `delegatecall`, `Chal` writes `latestReferrer` to slot 0, replacing the proxy’s implementation address with the helper.
3. Send any call to the proxy. Its fallback delegates to the helper, which self-destructs in the proxy’s context and transfers the proxy’s balance to you.
4. Select **3 — get flag** in the challenge spawner.

The drain transaction succeeded, and `Setup.isSolved()` returned `true`.

**Flag:** `PP{st0r4g3_c0ll1s10n_r3f3rr4l::c9l-UPv2BUFO}`

---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
