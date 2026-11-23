El Clásico — GlacierCTF 2026 Write-up
Category: Web / Cryptography (zk-SNARK soundness)
Points: 50
Author: Onirique
Challenge: “Can you prove the impossible?”
Target: https://el-clasico.web.glacierctf.com
Status: Solved

Flag: gctf{cl4ss1c_glv_n0t_s0_cl4ss1c}

1. Reconnaissance
The web root returned 404 page not found. This was expected: the supplied Go source registers only GET /challenge and POST /submit.

GET /challenge supplies a random BLS12-381 G1 point A = (a, b).
POST /submit expects decimal values s, p_x, p_y and a hex-encoded Groth16 proof in proof_hex.
The archive includes challenge.go, go.mod, pk.hex, and vk.hex; go.mod pins gnark v0.16.4.
The circuit performs BLS12-381 arithmetic inside a Groth16 circuit proved over BN254. This distinction matters: a weakness in the emulated curve gadget can make an invalid elliptic-curve computation satisfy the proof system's constraints.
The challenge verifies a public scalar s, a subgroup point P, and the public target A. It computes:

q := curve.ScalarMul(&c.P, &c.S)
dx := fp.Sub(&q.X, &c.A.X)
dy := fp.Add(&q.Y, &c.A.Y)
fp.AssertIsDifferent(&q.X, &c.A.X)
fp.AssertIsEqual(fp.Mul(fp.Mul(dx, dx), dx), fp.Mul(dy, dy))
Hence the required equation is

[ (q_x-a)^3 \equiv (q_y+b)^2 \pmod p, ]

with q_x != a, where q should equal the genuine scalar product [s]P.

The server also rejects s = 0, values out of range, points failing the subgroup checks, and scalars divisible by 9.

2. Vulnerability: unsound classic-GLV scalar multiplication
The pinned gnark release is affected by GHSA-4g5f-3gmm-9q87 (fixed in v0.16.5). BLS12-381's sw_emulated ScalarMul uses the classic GLV optimization. A hint splits a scalar into signed components satisfying

[ s_1+\lambda s_2 \equiv s\pmod r, ]

where lambda is the GLV eigenvalue and r is the subgroup order. The circuit checks the relation and bounds, but an attacker-controlled, valid-looking decomposition can make the optimized doubleAndAdd routine encounter an exceptional addition of equal points. Its incomplete addition formula assumes the points are distinct. When that assumption fails, a slope constraint becomes vacuous and does not fully bind the output to the genuine group result.

This is a soundness bug: the generated Groth16 proof can verify even when the intermediate scalar-multiplication result is mathematically wrong. We did not need to break Groth16 or recover a private key.

3. Choosing a scalar that passes the challenge filter
Let

lambda = 228988810152649578064853576960394133503
s1     = 2*lambda - 3
s2     = 7
s      = s1 + lambda*s2 = 9*lambda - 3
m      = 3*lambda - 1
Then s = 3m and s mod 9 = 6, so the input is not rejected by the “multiple of 9” rule. The malicious hint supplies s1, s2 and positive sign flags instead of the usual decomposition.

Given the public target point A, choose

[ P=[m^{-1}\bmod r]A. ]

This ensures that the genuine computation would yield [s]P = [3m]([m^{-1}]A) = [3]A. More importantly, the tailored GLV decomposition makes an internal exceptional addition occur at the known point A.

4. Making the forged output satisfy the final equation
Write A=(a,b) in the BLS12-381 base field. At the exceptional operation, the witness can induce a wrong output having the form

[ \ell=\frac{-2b}{3a}\pmod p,\quad q_x=a+\ell^2,\quad q_y=\ell^3-b. ]

The circuit's final check then holds automatically:

[ (q_x-a)^3=(\ell^2)^3=\ell^6, ]

[ (q_y+b)^2=(\ell^3)^2=\ell^6. ]

As long as ell != 0, the required q_x != a condition also holds. The exploit checks this before proving.

The important distinction is that this q is a consequence of the vulnerable circuit arithmetic, not the genuine [s]P.

5. Proof generation and submission
The exploit was implemented in Go using the exact ChallengeCircuit from the archive. It:

... (41 lines left) 

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
