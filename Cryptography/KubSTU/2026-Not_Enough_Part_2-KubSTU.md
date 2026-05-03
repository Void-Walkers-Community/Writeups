# Not enough part 2

## Challenge summary

The handout contains two RSA-like dumps and one final AES-GCM ciphertext.

- `dump_b` is a standard "known MSBs of one prime" recovery.
- `dump_a` looks similar at first, but the printed modulus is inconsistent with the printed prime prefix.
- After recovering both private exponents, the final AES key is derived from them through SHA-256.

The final flag is:

`KubSTU{1_h0p3_y0u_solv3d_7hi5_p4rt2_th1s_1s_much_h4rd3r}`

## 1. Establishing what `secret` means

Part 1 is useful as a calibration step. Recovering the missing low bits of `p` gives a valid RSA key, and testing the obvious candidates against the provided AES-GCM blob shows that the "secret" is the private exponent `d`.

So for part 2 the intended secrets are:

- `secret1 = d1`
- `secret2 = d2`

and the final KDF line is the usual truncation:

```python
key = sha256(long_to_bytes(d1) + long_to_bytes(d2)).digest()[:16]
```

## 2. Solving `dump_b`

For `dump_b`, the corrupted warning line clearly implies `80 low bits lost`.

Let

```text
p = p_hi * 2^80 + x,   0 <= x < 2^80
```

and solve the usual Coppersmith instance on

```text
f(x) = p_hi * 2^80 + x  (mod n)
```

This recovers:

```text
p2 = 10062055162138924214874824847807196795286955126475026299496126196298557795094160854981099370247925487576197518465890392600093676556658169963127903963728563
q2 = 10702907690154511017854937111874775924977478084094640245603510758714645942592948520012503707969619462265381655541832814234527211077932099308308287837299737
d2 = 78382713722957466927651103811638108297157801963061085704210440640082767002507653525600141289039604772731892403803914016357071803848166960855918853551978465567665175659338610360707545495164796735722313839469749679452660186852628767089446626920077299095206597991570769861255669161068935548172227267627600820673
```

## 3. Why `dump_a` does not factor as printed

`dump_a` claims that only the low `72` bits are missing, so we expect

```text
p = p_hi * 2^72 + x,   0 <= x < 2^72.
```

But the printed modulus does not fit that model:

- direct Coppersmith on the printed `(n, hint)` fails
- the printed `n` is not even semiprime
- the printed `hint` is nevertheless internally consistent as a true 440-bit prefix of a 512-bit prime

So the natural conclusion is that the `hint` is correct and `n` was corrupted during the dump.

## 4. Reducing the consistency check to a 2D CVP

Write

```text
p = H * 2^c + x
q = (Q + s) * 2^c + y
```

where:

- `H` is the recovered high part of `p`
- `c = 72`
- `Q = (n // (H * 2^c)) >> c`
- `s in {-1, 0, 1}` accounts for the fact that `Q` can differ from `q >> c` by at most 1
- `0 <= x, y < 2^c`

Let

```text
C = n - (H * 2^c) * (Q * 2^c)
L = C >> c
k = floor(xy / 2^c)
```

Then

```text
(Q + s) * x + k ≡ L - s * (H * 2^c) (mod H)
```

with both unknowns bounded by `2^c`:

```text
0 <= x < 2^c
0 <= k < 2^c
```

That turns candidate validation into a tiny 2D closest-vector problem instead of a full lattice factorization every time. Once `x` is known, recover

```text
y = (n mod 2^c) * x^{-1} mod 2^c
```

and verify `p * q == n`.

This exact test is fast enough to scan many decimal repairs of the printed modulus.

## 5. Repairing `dump_a`

Scanning two-digit decimal substitutions in the printed modulus with the CVP test finds the unique valid correction.

The corrupted slice was:

```text
...1362303666613385...
```

The corrected slice is:

```text
...1362300366613385...
```

Equivalently, zero-based decimal positions `234` and `235` changed from `36` to `03`.

So the corrected modulus is:

```text
82903356807644427511291773761378922029422611188358047221553242122036418386827855049155897729546477938247313902660497920739772848211750220810305389462531416989902465358554269027958064039184653500900006367700415049531522135923953913623003666133850180216800164771855523020348739786879677601906911639097689182081
```

Now the factorization is valid and the given high bits match exactly:

```text
p1 = 11682934587822315911375777283638722742905841461650629295792009964966349336971698849336863242750472746239661943288355824449431578999081220550343245425377599
q1 = 7096107248093178561280921148906200357200763243065263189174354891958252150330995489200695592491951762743329711594593658294321655850797436672577632033472319
d1 = 31049069881801615473162588873049508540399743528361814990814720354690997288626115052283923734252686283996525927961333620162011147281038332083997524823501734426134460681445636845332677037805359233150406320759657047031841518713245082399928064770644699148752415533134331912345787458165803995714842143960137838213
```

and

```text
p1 >> 72 == hint
```

is true.

## 6. Final decryption

Use the same convention as part 1:

```python
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util.number import long_to_bytes

key = SHA256.new(long_to_bytes(d1) + long_to_bytes(d2)).digest()[:16]
pt = AES.new(key, AES.MODE_GCM, nonce=nonce).decrypt_and_verify(data, auth)
```

This decrypts to:

```text
KubSTU{1_h0p3_y0u_solv3d_7hi5_p4rt2_th1s_1s_much_h4rd3r}
```

## 7. Takeaways

- `dump_b` is the straightforward intended warm-up: known-MSB RSA factor recovery.
- `dump_a` adds a second layer: the printed modulus is corrupted, so a normal Coppersmith attempt fails for a good reason.
- Rewriting the problem in terms of `x`, `y`, and `k = floor(xy / 2^c)` gives a very fast exact validator for candidate repairs.
- Once the corrected modulus is found, the rest is normal RSA reconstruction and AES-GCM decryption.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
