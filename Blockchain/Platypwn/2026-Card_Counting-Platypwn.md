# Card Counting writeup

The goal is to reduce the casino’s balance below 10 ETH. Roulette pays 36 times the bet when the selected number matches the result, so six winning bets at the maximum 0.5 ETH bet are enough.

## Recover the dealer’s RNG

The dealer uses Python’s `random.Random()` for both blackjack cards and roulette results. Each hand starts with one `getrandbits(32)` call, then converts that number into six base-52 card values:

```python
number = rng.getrandbits(32)
cards = []
for _ in range(6):
    number, card = divmod(number, 52)
    cards.append(card)
```

The six values reveal the original number:

```python
number = sum(card[i] * 52**i for i in range(6))
```

I played 625 hands and collected their cards from the `HandDealt` events. Those outputs were enough to reconstruct the MT19937 state. I verified the clone against the 625th output before using it.

## Predict roulette

With the RNG state synchronized, I predicted each `randrange(37)` result, then placed a 0.5 ETH bet on that number. The dealer’s settlements matched the predictions. After each win, I called `cashOut()` to transfer the winnings.

After six wins, the vault was drained to 0 ETH, satisfying `Setup.isSolved()`.

**Flag:** `PP{c0unt1ng_tw1st3d_c4rds::JS5SseqQostW}`



---
* [🔙 Back to Blockchain Directory](../)
* [🔙 Back to Blockchain Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
