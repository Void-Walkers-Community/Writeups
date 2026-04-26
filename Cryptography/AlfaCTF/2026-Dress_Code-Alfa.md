# Dress code

Type: web / crypto

## Summary

The app is a Flask shop backed by a Rust "validator" worker. Purchases are stored in the database as:

- `iv`
- `ciphertext`
- `mac`

The critical bug is that the HMAC covers only `ciphertext`, not `iv`. That makes the first plaintext block malleable in CBC mode.

There is also an authenticated SQL injection in [dresscode/shop/app.py](/home/kali/ctf-orc/ctf/dress-code/dresscode/shop/app.py:510):

```python
query = f"UPDATE transactions SET comment = '{new_comment}' WHERE id = '{order_id}'"
```

The validator logic in [dresscode/validator/src/main.rs](/home/kali/ctf-orc/ctf/dress-code/dresscode/validator/src/main.rs:131) verifies the MAC on ciphertext only, then decrypts using the stored IV. For purchases, if `from == "1000001"` it skips balance deduction.

## Exploit

1. Register accounts until the generated 7-digit user ID ends with `1`.
2. Add the 8 required owner-style items to the cart and create a purchase order.
3. Use the SQL injection in `update_comment` to rewrite the transaction `iv`.
4. Flip the first 6 digits of the JSON field `"from": "<our_id>"` into `"from": "1000001"`.
5. The MAC still validates because ciphertext is unchanged.
6. The validator processes the order as if it came from the owner account and gifts the full outfit to our user.
7. Visit `/check_dresscode` and read the flag.

Only the first CBC block needs modification. With an ID like `3486481`, the plaintext starts as:

```text
{"from": "3486481", ...
```

The final digit is already `1`, so the IV patch only changes the first 6 digits to turn it into:

```text
{"from": "1000001", ...
```

## Result

Flag:

```text
alfa{5mO7rYA_KAK01_fAbRic_5m0trY4_skolk0_d3TAIl5}
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
