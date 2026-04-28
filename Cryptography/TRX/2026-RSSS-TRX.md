# RSSS — CTF Writeup

**Event:** The Roman Xpl0it CTF  
**Category:** Cryptography  
**Author:** fwame  
**Flag:** `TRX{b4d_1d34_w0r53_3x3cut10n}`

---

## The Challenge

The server has a **secret polynomial** with 32 random coefficients. The polynomial looks like this:

```
f(x) = c₀ + c₁·x + c₂·x² + ... + c₃₁·x³¹  (mod p)
```

The secret we need is `c₀` (the first coefficient, also called `poly[0]`).

We can ask the server up to **16 questions**. Each time, we send a number `x`, and the server tells us `f(x) mod p`. At the end, if we can give the correct value of `c₀`, we get the flag.

The server also gives us the value of `p`.

---

## First Look at the Code

```python
poly = [secrets.randbelow(p) for _ in range(32)]  # 32 random coefficients

for i in range(16):
    x = int(input("> ")) % p
    if x == 0:
        break
    print(sum([c * pow(x, i, p) for i, c in enumerate(poly)]) % p)

x = int(input("> ")) % p
if x == poly[0]:
    print(flag)
```

The polynomial has **degree 31** (32 coefficients). To fully reconstruct a degree-31 polynomial, you normally need **32 points**. But we only get 16 queries. This looks impossible at first.

---

## The Key Question

> Is `p` actually a prime number?

The code treats `p` like a prime, but it never checks this. Let's test it:

```python
from sympy import isprime, factorint

p = 74199240114010686972283686524130048970447653679144743487258240101613967110409
print(isprime(p))   # False!
print(factorint(p))
```

**Result:**
```
False
{
  23: 1,
  491: 1,
  2459869: 1,
  629537976415099919: 1,
  4242835718584040738005042668915821379247204089583: 1
}
```

**`p` is not prime!** It is a product of 5 prime numbers:

```
p = 23 × 491 × 2459869 × 629537976415099919 × 4242835718584040738005042668915821379247204089583
```

This is the whole vulnerability.

---

## The Math Trick

When we query at `x = q` where `q` is a **prime factor of `p`**, something special happens.

The polynomial evaluation is:

```
f(q) = c₀ + c₁·q + c₂·q² + ... + c₃₁·q³¹  (mod p)
```

Now, let's think about what happens **mod `q`**:

- `q ≡ 0 (mod q)`
- `q² ≡ 0 (mod q)`
- `q³ ≡ 0 (mod q)`
- ... and so on for all powers

So every term **except `c₀`** disappears:

```
f(q)  ≡  c₀  (mod q)
```

The server returns `f(q) mod p`. But if we then take that result **mod q**, we get `c₀ mod q`.

**In simple words:** querying at `x = q` gives us one "piece" of `c₀`.

---

## Chinese Remainder Theorem (CRT)

We have 5 prime factors. With 5 queries (one per factor), we get:

| Query | What we learn |
|-------|--------------|
| `x = 23` | `c₀ mod 23` |
| `x = 491` | `c₀ mod 491` |
| `x = 2459869` | `c₀ mod 2459869` |
| `x = 629537976415099919` | `c₀ mod 629537976415099919` |
| `x = 4242835...` | `c₀ mod 4242835...` |

**CRT** says: if we know `c₀` modulo several coprime numbers, we can find the unique `c₀` modulo their product. Since the product of all 5 factors equals `p`, and `c₀ < p`, we recover `c₀` exactly.

---

## The Solve Script

```python
from functools import reduce
from pwn import remote

FACTORS = [
    23,
    491,
    2459869,
    629537976415099919,
    4242835718584040738005042668915821379247204089583,
]

def crt(residues, moduli):
    M = reduce(lambda a, b: a * b, moduli)
    result = 0
    for r, m in zip(residues, moduli):
        Mi = M // m
        result += r * Mi * pow(Mi, -1, m)
    return result % M

io = remote("rsss.ctf.theromanxpl0.it", 9097)

# Read p from server
line = io.recvline().decode().strip()
p = int(line.split("=")[1].strip())

# 5 queries — one per factor
residues = []
for q in FACTORS:
    io.recvuntil(b"> ")
    io.sendline(str(q).encode())
    y = int(io.recvline().decode().strip())
    residues.append(y % q)

# Recover c₀ with CRT
c0 = crt(residues, FACTORS)

# End the query loop, then submit c₀
io.recvuntil(b"> ")
io.sendline(b"0")
io.recvuntil(b"> ")
io.sendline(str(c0).encode())

print(io.recvline().decode().strip())
io.close()
```

---

## Output

```
[+] Opening connection to rsss.ctf.theromanxpl0.it on port 9097: Done
[*] p = 74199240114010686972283686524130048970447653679144743487258240101613967110409

    f(23) mod p             = 66238658145796821553773340647131091225715653648992419901694529092299560591897
    c₀ mod 23               = 14

    f(491) mod p            = 57027215259886459264241344091064870115552051739616560366051724237308942845647
    c₀ mod 491              = 481

    f(2459869) mod p        = 31320320613745349428818049962841549536978884837414311059815645066142643432846
    c₀ mod 2459869          = 2454422

    f(629537976415099919) mod p = 50541551824595047072938721383727098362664470074411345470388751636232186104459
    c₀ mod 629537976415099919   = 195361431830295646

    f(4242835...89583) mod p    = 52838525365709889455276819934405056193703728561441294522476091802905015968276
    c₀ mod 4242835...89583      = 1391446539624019383841064387497432895133140316094

[+] Recovered poly[0] = 64627351264747900771559469929430273676490140522655417959326531806551079317763
🚩 FLAG: TRX{b4d_1d34_w0r53_3x3cut10n}
```

---

* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
