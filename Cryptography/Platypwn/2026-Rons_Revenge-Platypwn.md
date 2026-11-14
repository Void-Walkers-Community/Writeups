# Ron's Revenge — Writeup

## Challenge

The service implements a parameterized RC4 variant with a 2048-entry state. The key is

```text
IV (3 chosen values) || longterm_key (8 secret values)
```

where `longterm_key` is generated once from `random.seed(flag)` and reused for every query. The service repeatedly encrypts the same flag under different chosen IVs:

```python
cipher = Stream(iv + longterm_key, 2048)
ciphertext = cipher.encrypt(flag)
```

The flag format is `PP{...}`, so the first two plaintext bytes are known to be `PP`.

## The RC4 variant

`RC4_1` is standard RC4 with state size `N = 2048`:

```python
j = 0
for i in range(N):
    j = (j + S[i] + key[i % len(key)]) % N
    S[i], S[j] = S[j], S[i]
```

The PRGA produces `output_bits = log2(N) = 11` bits per step:

```python
a = (a + 1) % N
b = (b + S[a]) % N
S[a], S[b] = S[b], S[a]
output = S[(S[a] + S[b]) % N]
```

`Stream` concatenates these 11-bit values and emits bytes. Therefore the first keystream byte is the top 8 bits of the first 11-bit output, and the remaining 3 bits are the top 3 bits of the second keystream byte:

```text
ks[0] = z0 >> 3
ks[1] = ((z0 & 7) << 5) | (z1 >> 6)
```

Because `flag[0] = flag[1] = 'P'`, each oracle query gives us the **exact first 11-bit PRGA output**:

```python
z0 = ((ct[0] ^ ord('P')) << 3) | ((ct[1] ^ ord('P')) >> 5)
```

So the unknown part is only the 8 secret key values, each in `[0, 2047)`.

## Recovering the key bytes

Let `K = IV || k3 || k4 || ... || k10`, where `k_m` is the secret key value at overall key index `m`. We recover `k3, k4, ..., k10` sequentially.

### First PRGA output

After the KSA, the first PRGA step uses `a = 1` and `b = S[1]`. The swap does not change the sum of the two values, so the output is read from

```text
index = (S[1] + S[S[1]]) mod N
```

Suppose we choose an IV such that, after KSA step `m-1`,

```text
S[1] + S[S[1]] ≡ m   (mod N)
```

Then the first PRGA output reads position `m` after KSA step `m`.

At KSA step `m`:

```text
j_m = (j_{m-1} + S[m] + k_m) mod N
```

The swap moves the old value `S[j_m]` into position `m`. If `j_m` is not one of the PRGA pointer positions `1` or `S[1]`, the first PRGA output becomes

```text
z = old S[j_m]
```

or `old S[m]` when `j_m = m`.

Since the state immediately before step `m` can be computed from the IV and the already recovered prefix `k3 ... k_{m-1}`, we can invert the observation for a given IV:

```text
j_m        = position of value z in the pre-step-m state
candidate  = (j_m - j_{m-1} - S[m]) mod N
```

Each IV gives one vote for a candidate `k_m`. The correct candidate receives votes whenever the remaining KSA steps do not disturb the relevant state positions; invalid queries behave like random votes. Aggregating many IVs makes the true key value stand out clearly.

### Generating useful IVs

The first `m-1` KSA steps touch only a few state positions, so they can be simulated with a sparse dictionary instead of a full 2048-element array.

To make the condition `S[1] + S[S[1]] ≡ m` common, the generator uses a family of state-shaping IVs. For a chosen `t` in `2 ... m-1`:

1. Pick `iv0` randomly.
2. Set `iv1 = (m - t - 1 - iv0) mod N`, so step 1 attempts to put `m-t` into `S[1]`.
3. Scan `iv2`.
4. Simulate the first `m` KSA steps and keep only IVs satisfying `S[1] + S[S[1]] ≡ m`.

The final condition is checked against the real sparse simulation, so collisions and edge cases are naturally filtered out.

### Recovery loop

For `m = 3 ... 10`:

1. Generate useful IVs with the known prefix.
2. Query the oracle for each IV.
3. Extract the exact first PRGA output `z0` from the first two ciphertext bytes.
4. Vote using the formula above.
5. Take the most frequent candidate and append it to the known prefix.

The vote counts obtained against the live server were:

| key index | recovered value | top votes | runner-up |
|-----------|-----------------|-----------|-----------|
| `k3`      | 1531            | 10        | 2         |
| `k4`      | 1602            | 21        | 2         |
| `k5`      | 313             | 20        | 8         |
| `k6`      | 1066            | 27        | 5         |
| `k7`      | 124             | 64        | 13        |
| `k8`      | 1152            | 61        | 7         |
| `k9`      | 1206            | 78        | 6         |
| `k10`     | 480             | 88        | 10        |

Recovered long-term key:

```text
[1531, 1602, 313, 1066, 124, 1152, 1206, 480]
```

## Decryption

With the full key known, decrypt any stored ciphertext. Using IV `(0, 0, 0)`:

```python
key = [0, 0, 0] + longterm_key
plaintext = Stream(key, 2048).encrypt(ciphertext)
```

This yields:

```text
PP{st4t3s1z3_w4s_n0t_th3_m41n_pr0bl3m::QBQQ7fW-qXLy}
```

As a consistency check, re-encrypting this plaintext with the recovered key under IV `(0,0,0)` reproduces the server ciphertext exactly.

## Flag

```text
PP{st4t3s1z3_w4s_n0t_th3_m41n_pr0bl3m::QBQQ7fW-qXLy}
```

## Solver highlights

```python
def sparse_steps(iv, nsteps, prefix):
    """Simulate the first nsteps of the KSA using only touched positions."""
    d = {}
    def S(i):
        return d.get(i, i)
    key = list(iv) + list(prefix)
    j = 0
    for i in range(nsteps):
        si = S(i)
        j = (j + si + key[i % 11]) % N
        sj = S(j)
        d[i] = sj
        d[j] = si
    return d, j


def pos_of_value(d, v):
    for k, val in d.items():
        if val == v:
            return k
    return v


def gen_valid(m, prefix, niv0, seed):
    """Generate IVs for which the first PRGA output reads state position m."""
    rng = random.Random(seed)
    seen, out = set(), []
    for t in range(2, m):
        for _ in range(niv0):
            iv0 = rng.randrange(N)
            iv1 = (m - t - 1 - iv0) % N
            for iv2 in range(N):
                iv = (iv0, iv1, iv2)
                if iv in seen:
                    continue
                d, jprev = sparse_steps(iv, m, prefix)
                b = d.get(1, 1)
                vb = d.get(b, b)
                if (b + vb) % N != m or b == m or b == 1:
                    continue
                seen.add(iv)
                out.append((iv, jprev, d.get(m, m), d))
    return out
```

Voting for one key byte:

```python
votes = collections.Counter()
for (iv, jprev, Sm, d), ct in zip(valid, ciphertexts):
    ks0 = ct[0] ^ ord('P')
    ks1 = ct[1] ^ ord('P')
    z = (ks0 << 3) | (ks1 >> 5)

    jm = pos_of_value(d, z)
    b = d.get(1, 1)
    if jm in (1, b):
        continue
    votes[(jm - jprev - Sm) % N] += 1

k_m = votes.most_common(1)[0][0]
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
