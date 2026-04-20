# Baby Exponent Writeup

## Challenge

We are given an RSA-style encryption challenge:

```text
n = 3975111046...
e = 3
c = 2120801644...
```

The challenge title is **Baby Exponent**, which hints that the RSA public exponent is very small.

## Vulnerability

In RSA, encryption is:

```text
c = m^e mod n
```

Here, the exponent is:

```text
e = 3
```

Usually, the modulo operation protects the plaintext because `m^e` wraps around modulo `n`.

However, if the plaintext integer `m` is small enough such that:

```text
m^3 < n
```

then the modulo operation does nothing.

So instead of:

```text
c = m^3 mod n
```

we simply have:

```text
c = m^3
```

That means we can recover the plaintext by taking the integer cube root of `c`.

## Exploit Plan

1. Read the given ciphertext `c`.
2. Compute the integer cube root of `c`.
3. Convert the resulting integer back into bytes.
4. Decode the bytes to get the flag.

## Solver

```python
from Crypto.Util.number import long_to_bytes
import gmpy2

n = int("""
39751110465815367049531864518769782848382242730514875914573008861502728
99565288847783297896686373864893218348554640229201785045236064536514210026
833637120465988737155155159875330523198560124610157483399935625056352106495
6134365407699223
""".replace("\n", ""))

e = 3

c = int("""
21208016443347524194488872231478291493949438395584503771520814766894326694
962664570764050936260992180345927690604412742209707097487410379538181314694
35699367735940032724483543045224740051080037
""".replace("\n", ""))

m, exact = gmpy2.iroot(c, 3)

if not exact:
    print("Cube root was not exact. Attack may not work.")
else:
    flag = long_to_bytes(int(m))
    print(flag.decode())
```

## Output

```text
CIT{sm4ll_3xp0n3nt_g0_brrr}
```

## Flag

```text
CIT{sm4ll_3xp0n3nt_g0_brrr}
```

## Why This Works

This is a classic low-exponent RSA mistake.

RSA with a small public exponent like `e = 3` is not automatically broken, but it becomes vulnerable when the same plaintext is encrypted directly without padding and the plaintext is small enough.

Secure RSA encryption should use randomized padding such as OAEP. Without padding, small messages can be recovered directly when:

```text
m^e < n
```

In this challenge, because `e = 3`, we only needed to calculate:

```text
m = ∛c
```

Then converting `m` from a long integer to bytes reveals the flag.


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
