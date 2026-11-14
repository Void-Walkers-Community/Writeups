# PlatyPals writeup

The vault starts with 100 ETH. `Setup.isSolved()` checks whether the vault balance is below 10 ETH. Each box costs 1 ETH, while selling a legendary returns 50 ETH.

The backend opens boxes in batches. If a recipient contract’s `onPalReceived` callback reverts, the entire `openBoxes` transaction reverts. The backend then retries unopened boxes with fresh random rolls.

## Exploit

Deploy a receiver contract that:

1. Buys one box from `Chal`.
2. Reverts in `onPalReceived` unless the rarity is `LEGENDARY`.
3. When it receives a legendary, calls `sellLegendary()` from the callback.
4. Lets the backend retry the same box until a legendary roll succeeds.

Buy and open two boxes this way. The vault balance becomes:

`100 + 2 × 1 − 2 × 50 = 2 ETH`

That is below the 10 ETH solve threshold.

**Flag:** `PP{r3r0ll_unt1l_l3g3nd4ry::38nnZEO-vMkQ}`


---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
