# Short Writeup: Not Enough Part 1
We are given:
- RSA modulus `N`
- Public exponent `e = 65537`
- The upper `440` bits of a `512`-bit prime `p` as `p_hi`
- AES-GCM parameters: `nonce`, `ciphertext`, `tag`
- KDF: `sha256(long_to_bytes(secret)).digest()[:16]`
Since `p` is 512 bits and only its last `72` bits are missing, we write:
```text
p = (p_hi << 72) + x
```

where `x < 2^72`.

Because `x` is small, this is a standard small-root / Coppersmith-style situation. Recover `x`, rebuild `p`, compute:

```text
q = N // p
phi = (p-1)(q-1)
d = e^-1 mod phi
```

Then test the recovered RSA secrets against the given KDF. The correct secret is `d`, so the AES key is:

```python
sha256(long_to_bytes(d)).digest()[:16]
```

Decrypting the AES-GCM ciphertext gives the flag:

```text
KubSTU{1_h0p3_y0u_solv3d_7hi5_wi7h0ut_4ny_pr0bl3m5}
```

---
## Script

```python
from sympy import symbols, Poly, Matrix
from Crypto.Util.number import long_to_bytes, inverse
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from hashlib import sha256

N = 151708532784988710186354895816447243710932251919277531742510058529452761722432439526454251312377007965942929512494581288744504881159769296873653469521440122432931073443340101324374399582714454128949856004044525971541673007542812412410730741409339131837689130677766341334202879471841791194383321275827852931391
e = 65537
p_hi = 2615850379731327725778203313365784512838922702185011257029931947846122736769634242897120915009678837272252972592332233245419497072070

nonce = bytes.fromhex("8fe6c8d25d0738576b6f6a25")
ciphertext = bytes.fromhex("0094dfe5f358aecb96369cf72731d114bf0a0008cbe1d15b98b30f4fd1492e0ee1567a7fd602dc3ff7aa709ea98e7c06eb261c")
tag = bytes.fromhex("9755d120d8356f29ca31eacff3360ab0")

# p = P + x, where x is the missing 72-bit suffix
P = p_hi << 72
X = 2**72
x = symbols("x")
f = Poly(x + P, x, domain="ZZ")

# Small-root lattice setup
m = 2
t = 2
fX = Poly(f.as_expr().subs(x, x * X), x, domain="ZZ")

polys = []
for i in range(m):
    polys.append(Poly((N ** (m - i)) * (fX ** i), x, domain="ZZ"))
for i in range(t):
    polys.append(Poly((x * X) ** i * (fX ** m), x, domain="ZZ"))

maxdeg = max(p.degree() for p in polys)
rows = [[int(p.nth(k)) for k in range(maxdeg + 1)] for p in polys]
M = Matrix(rows)
B = M.lll()

root = None
for ridx in range(B.rows):
    row = B.row(ridx)
    coeffs = [int(row[i] // (X ** i)) for i in range(maxdeg + 1)]
    g = Poly(sum(c * x**i for i, c in enumerate(coeffs)), x, domain="ZZ")
    for r in g.ground_roots():
        r = int(r)
        if 0 <= r < X:
            cand_p = P + r
            if N % cand_p == 0:
                root = r
                break
    if root is not None:
        break

if root is None:
    raise ValueError("Failed to recover missing bits")

p = P + root
q = N // p
phi = (p - 1) * (q - 1)
d = inverse(e, phi)

key = sha256(long_to_bytes(d)).digest()[:16]
pt = AESGCM(key).decrypt(nonce, ciphertext + tag, None)

print(pt.decode())
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
