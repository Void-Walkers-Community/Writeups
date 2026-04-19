Idea
This is a Groth16 verifier with the verification key hardcoded in challenge_params.py.  
So instead of proving a real witness, you can forge a proof that satisfies the pairing equation directly.
Verifier equation
From server.py:
e(B, A) = e(beta, alpha) * e(gamma, ic) * e(delta, C)
where ic = gamma_abc[0] + out * gamma_abc[1].
Forge
Pick:
- A = alpha
- B = beta
- C = -gamma / delta * ic in the BN128 scalar field
Let k = (-GAMMA * DELTA^-1) mod CURVE_ORDER, then C = [k]ic.
Then:
e(delta, C) = e(delta, [k]ic) = e(gamma, ic)^-1
so the RHS becomes:
e(beta, alpha) * e(gamma, ic) * e(gamma, ic)^-1 = e(beta, alpha)
which matches the LHS exactly.
Why it works
- The public output out changes every round, but only affects ic.
- Since the VK is public, the proof can be recomputed per round.
- No secret witness is needed.
Core solve logic
k = (-GAMMA * pow(DELTA, -1, CURVE_ORDER)) % CURVE_ORDER
C = k * ic
proof = {
    "A": ALPHA_G1,
    "B": BETA_G2,
    "C": point_to_json(C),
}
Repeat for all 32 rounds, and the server prints the flag.
Flag
squ1rrel{zksn4rk_s4ys_4rtem1s_ii_n3v3r_h4pp3n3d_3ith3r}
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
