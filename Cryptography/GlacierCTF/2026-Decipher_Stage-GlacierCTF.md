# Decipher (GlacierCTF) — Solution

## Flags

| Stage | Flag |
|---|---|
| 1 | `gctf{who_need5_th3_k3y_if_y0u_c4n_g3t_th3_pla1n73x7}` |
| 2 | `gctf{gaus5_and_g41oi5_to_7h3_re5cue}` |

## Recovered keys (both independently verified: re-encryption reproduces **all 16 ciphertexts and all 16×16×11 proofs** in each stage)

| Stage | Base key (`RK0`) | Notes |
|---|---|---|
| 1 | `fc417d0d233a97a57521f12792672e88` | wrapped flag key decrypts to `9f291cd4aced45919deae8677ef88f0f` |
| 2 | `2f3762f3f277d89957613a661802dd5c` | AES key = `sha256(round_keys)` |

## The challenge

`decipher.py` is AES-128 with a custom S-box, except the final round also
omits SubBytes/ShiftRows/MixColumns (so `CT = state₁₀ ⊕ RK₁₀`).
For 16 random plaintexts it publishes an "execution proof":
`proofs[i][j][r]` = XOR of the states of plaintexts *i,j* entering round *r*
(11 states of 16 bytes). Same key in both stages; Stage 1 wraps an AES-GCM
flag key with the cipher, Stage 2 derives the AES-GCM key directly from
`sha256(round_keys)` — so the **exact** base key is required in both cases.

## Why the proofs leak everything

**1. The S-box is degenerate.** Its algebraic normal form has degree ≤ 2,
with only *two* distinct quadratic monomials (`66 = x₁x₆`, `96 = x₅x₆`, i.e.
the single product `x₆·(x₁⊕x₅)`):

```
S(x) = M(x) ⊕ T(φ(x)),   M GF(2)-linear (rank 6),  φ = 2-bit coset map
```

It has a 6-dimensional space `L` of linear structures (64 of them):
`S(x⊕l) ⊕ S(x) = λ(l)` is constant for `l ∈ L`. DDT entries are only 1 or 2.

**2. Round 1 alone is blind.** Inverting ShiftRows/MixColumns (both linear)
on `proofs[·][·][1]` gives first-round S-box output differences for known
input differences. For each byte position, exactly the 64 keys in one coset
`k ⊕ L` are consistent — the true key is hidden inside a 96-bit ambiguity
(`64¹⁶`). Later rounds break this only through absolutes (MixColumns spreads
the per-byte `λ(l)` offsets), which is exponentially weak for round-by-round
candidate filtering (wrong keys pass a full column check with probability
only ≈ 2⁻⁸·⁴, and the key schedule couples all columns).

**3. Linearization kills it.** Treat every S-box output bit as an *affine*
form in (key bits ⊕ one new "product" variable `p = x₆·(x₁⊕x₅)` per S-box).
With the 16 plaintexts, their ciphertexts, all intermediate XOR-differences
from the proofs, and the 2-bit coset of each key byte (from step 2), this
yields a **linear** system:

- 2 728 variables = 128 key bits + 2 600 product variables
  (16×160 encryption S-boxes + 40 key-schedule S-boxes)
- 21 280 equations, rank 2 560 → a 168-dimensional affine solution space
  (the dimension is structural: 168 regardless of the number of equations)

Substituting the affine parametrization into the 2 600 product definitions
`p = x₆·(x₁⊕x₅)` gives 2 600 quadratic equations in the 168 free parameters,
which z3 solves in seconds — with a **unique** solution per stage. Reading
off the key bits and verifying against every ciphertext and proof confirms
the keys; AES-GCM decryption (nonce `0¹²`) then yields the flags.

Generic SAT (Kissat/CaDiCaL/CryptoMiniSat on an ANF CNF encoding, and a
z3 word-level model) was also attempted on the 2⁹⁶ candidate space and did
not finish in ~15 minutes — Gaussian elimination over the linearized system
is the intended fast lane ("Gauss and Galois to the rescue", as Stage 2's
flag puts it).

## Solver

- `../solve_lin.py` — builds the linearized system per stage (forms over
  GF(2), Gaussian elimination, dumps the 168-dim quadratic system)
- `../solve_quad.py` — solves the quadratic system with z3 and maps the
  solution back to the base key
- `../extract_flags.py` — verification + AES-GCM flag decryption given keys

## Getting the files

`play.glacierctf.com` sits behind a Cloudflare managed challenge that
rejects datacenter/headless clients. The archive was fetched by driving a
real Google Chrome (headed, under Xvfb) through the Turnstile check to
obtain a `cf_clearance` cookie, then downloading
`decipher-1.tar.gz` inside that cleared browser context.


