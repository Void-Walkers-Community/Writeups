# Poly-Crypto Writeup

## Challenge

- Name: `Poly-Crypto`
- Category: `Crypto`
- Points: `500`
- Author: `grdnero`

## Summary

The challenge provides three files:

- `artifact.png`
- `output.txt`
- `solve.py`

`output.txt` contains an RSA public modulus `n`, exponent `e`, and ciphertext `c`.  
`solve.py` shows that the private key was deterministically generated from two hidden byte strings:

```python
def derive():
    a = get_part_a()
    b = get_part_b()

    p = nextprime(int.from_bytes(sha512(b"P" + a).digest(), "big"))
    q = nextprime(int.from_bytes(sha512(b"Q" + b).digest(), "big"))
    return p, q
```

So the goal is not to factor RSA directly. The goal is to recover enough of the missing seed material to reconstruct one prime factor.

## Step 1: Inspect the PNG carefully

The visible image is just high-entropy noise, but metadata inspection reveals an important clue:

- The PNG contains trailing data after the `IEND` chunk.

This means `artifact.png` is a polyglot or carrier file.

The PNG ends at offset `787247`, and a PDF begins at offset `787275` with the header:

```text
%PDF-1.5
```

## Step 2: Carve the embedded PDF

Extracting the appended PDF from the PNG reveals a one-page document.  
That PDF contains two short hexadecimal-looking strings:

```text
9f2c6b7a1d84e3c
0f5a927d6bce41308
```

At first glance, both strings have odd length, which makes them look inconvenient as raw hex values.  
The key observation is that together they form exactly 32 hexadecimal characters:

```text
9f2c6b7a1d84e3c0f5a927d6bce41308
```

That is a clean 16-byte value.

## Step 3: Reconstruct `a`

Concatenate the two strings and hex-decode them:

```python
a = bytes.fromhex("9f2c6b7a1d84e3c0f5a927d6bce41308")
```

Now compute:

```python
p = nextprime(int.from_bytes(sha512(b"P" + a).digest(), "big"))
```

This produces a valid factor of `n`.

That is enough to solve the challenge because:

```python
q = n // p
```

Even without reconstructing the original `get_part_b()`, once one prime factor is known, RSA is broken.

## Step 4: Recover the private key and decrypt

With `p` and `q`, compute:

```python
phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)
```

Convert `m` back to bytes to recover the plaintext.

## Minimal Solve Script

```python
from hashlib import sha512
from sympy import nextprime

n = 17567450016852554887621588368698419645874283756013660732387924661550491278193597938535420719133766888159473988870244443505340840195041998570182396185220358645806449134357725419319260767055174486678285439225135830651475797137124428713076463187533478141381546346507894029119545930501022276653776873326710776167
e = 65537
c = 16627164671812233365513248669922004245110523221676426623094362348549422239042558413691341901150839168501387115550428949017040971899084355076790626544446696248914280516850881175335306668349001031973308285949557769269719125599564129066927042591488748147935928016976833686017096902513380336643162606787826272062

a = bytes.fromhex("9f2c6b7a1d84e3c0f5a927d6bce41308")

p = nextprime(int.from_bytes(sha512(b"P" + a).digest(), "big"))
q = n // p

assert p * q == n

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)

pt = m.to_bytes((m.bit_length() + 7) // 8, "big")
print(pt.decode())
```

## Recovered Values

```text
p = 11782828714201933570161460556962759538428773419018586701638526376713720180452202497367879990916071865558239317796753404850285921988154237784857846677124439
q = 1490936552075850259769522910240569706341090740514174705443054492738358515187211973280746310184804643381248823843861405602814007995108270379996686325783153
```

## Flag

```text
CTFAC{a8de7eec2b0d529d0c234098e6988a9133110a89af5080d0c465868a5794bbe2}
```

## Takeaway

This challenge is a good example of deterministic-key RSA being only as strong as the secrecy of its seed material.  
Once the embedded PDF leaked enough data to reconstruct `a`, the modulus no longer needed a hard factorization attack. A single recovered seed was enough to derive one prime, divide `n`, and decrypt the ciphertext.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
