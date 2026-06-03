**Writeup**

We are given:

```python
N = p^2 * q
e = 65537
c = pow(m, e, N)
p_msb = (p >> 320) << 320
```

So `p` is mostly known. Let:

```text
p = p_msb + x
```

where the unknown `x` is only the lower 320 bits, so:

```text
0 <= x < 2^320
```

Because `N = p^2 q`, we know:

```text
(p_msb + x)^2 ≡ 0 mod p^2
```

The unknown divisor `p^2` is large, about `N^(2/3)`, and the root `x` is small. This is exactly a Coppersmith small-root setup.

Use Sage:

```python
N = ...
e = 65537
c = ...
p_msb = ...

X = 2^320

PR.<x> = PolynomialRing(Zmod(N))
f = (p_msb + x)^2

roots = f.small_roots(X=X, beta=2/3)
x0 = int(roots[0])

p = p_msb + x0
q = N // (p*p)

phi = p * (p - 1) * (q - 1)
d = inverse_mod(e, phi)

m = pow(c, int(d), N)
flag = int(m).to_bytes((int(m).bit_length() + 7)//8, "big")
print(flag)
```

Recovered:

```text
p = 179147486404486085085422197280000587511454751621722835223057137715594698827830504944899819370021301435651195665075445171992202325618549266532203524209842043097900940030382430999458527271239232703644029719824618424028422796321821220454073981257342414462518827755228640188642932291796013989583309149316750465873
```

Then decrypting gives:

```text
grey{th1s_15_pr0b4bly_t00_34sy_n0w4d4y5_1n34v80n23}
```

---
* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
