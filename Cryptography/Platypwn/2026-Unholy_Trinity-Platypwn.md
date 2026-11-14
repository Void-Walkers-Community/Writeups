# Unholy Trinity — Crypto Writeup

The service uses RSA with public exponent \(e=17\). It hashes a password as

\[
H(m)=m^{17}\bmod N
\]

The server then uses that value in a challenge-response login and compares password hashes during registration. The two flag paths come from weaknesses in those operations.

## Flag 1: Recover the admin secret

The admin password is 19 random bytes. Its integer value \(s\) is at most 152 bits, so \(s^{17}\) is at most 2,584 bits. The RSA modulus is about 2,674 bits. Therefore, for the admin account:

\[
H=s^{17}
\]

There is no modular wraparound. During login, the server prints the admin’s committed password hash \(H\), so take its exact integer 17th root to recover \(s\).

For each login round, choose announcement \(a=1\). The server returns a challenge \(c\) between 0 and 16. Respond with:

\[
r=s^c
\]

Then:

\[
r^{17}=(s^c)^{17}=(s^{17})^c=H^c \pmod N
\]

That satisfies the server’s check. Repeat for all 52 rounds to log in as admin and receive flag 1.

## Flag 0: Register a hash collision

Registration requires a password of `NLEN` bytes, where `NLEN` is the byte length of \(N\). For this challenge, that is 335 bytes. The hash converts the entire password to an integer and reduces it modulo \(N\).

Choose two distinct 335-byte passwords whose integer values satisfy:

\[
m_2=m_1+N
\]

Then \(m_2\equiv m_1\pmod N\), so:

\[
m_2^{17}\equiv m_1^{17}\pmod N
\]

Register the first password, then register the second under a different username. The second registration detects the matching hash and prints flag 0.

To construct a pair, choose \(m_1\) so both \(m_1\) and \(m_1+N\) fit in 335 bytes and both passwords satisfy the character checks. Randomly sampling candidates works quickly.

## Flags

- **Flag 0:** `PP{modulo_hashing_is_not_collision_resistant::XHcsQCWXkQTz}`
- **Flag 1:** `PP{rsa_messages_need_to_be_sufficiently_large::DwC8Oftc0nmG}`


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
