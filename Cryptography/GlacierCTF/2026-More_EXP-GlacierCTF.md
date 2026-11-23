more_EXP — Writeup (GlacierCTF 2026)
Flag: gctf{a6f3cf9ce77fb0debba1f2852bbe652e9daabd0ae2cd57aada7a2fa3bffb2c25}

The setup
The service creates a single 4096-bit RSA modulus N = p·q (2048-bit primes). It's an encryption oracle: you send JSON with an exponent e (any e >= 65537) and a message msg, and it returns c = m^e mod N. If you omit msg, it encrypts the flag. Decryption isn't implemented, but the math makes that irrelevant.

Step 1 — Leak the modulus with m = -1
You never see N directly since outputs are reduced mod N. But with an odd exponent:

pow(-1, e, N) = N − 1   (for odd e)


Sending {"msg": -1, "e": 65537} returns N−1, so N = result + 1 (4095 bits on the live service).

Step 2 — Common-modulus attack on the flag
No padding is used and the modulus is reused for every query. Request the flag under two coprime exponents:

c1 = FLAG^65537 mod N
c2 = FLAG^65539 mod N


Both exponents pass the >= 65537 check, and gcd(65537, 65539) = 1. Bézout gives:

32769·65537 − 32768·65539 = 1


Therefore, using a modular inverse for the negative exponent:

m = c1^32769 · c2^(−32768) = FLAG^(32769·65537 − 32768·65539) = FLAG (mod N)


Decode the resulting integer to bytes and print the flag — no factoring, no private key, no decryption endpoint needed.

Key lesson
A single RSA modulus with unpadded messages and user-chosen exponents is broken by the common-modulus attack; requiring large exponents doesn't help, and allowing arbitrary messages leaks N for free

