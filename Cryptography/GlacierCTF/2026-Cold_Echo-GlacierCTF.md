# Cold Echo write-up

Work over the prime field \(\mathbb F_p\), where \(p=17179869143\). Write the unknown calibration profile as

\[
P(X)=a_0+a_1X+\cdots+a_{42}X^{42}.
\]

The trace readings are values of \(P(u)P(u^{-1})\). Define the reciprocal product

\[
Q(X)=P(X)P(X^{-1})
=q_0+\sum_{k=1}^{42}q_k(X^k+X^{-k}).
\]

This reduces the reconstruction to 43 unknowns \(q_0,\ldots,q_{42}\). Each trace gives one linear equation:

\[
q_0+\sum_{k=1}^{42}q_k(u^k+u^{-k})=\text{reading}\pmod p.
\]

The commissioning condition \(P(1)=1\) gives one more, since \(Q(1)=P(1)^2=1\):

\[
q_0+2\sum_{k=1}^{42}q_k=1\pmod p.
\]

Solving these 43 equations recovers \(Q\). Multiplying by \(X^{42}\) gives an ordinary degree-84 polynomial:

\[
R(X)=X^{42}Q(X)=P(X)\,X^{42}P(X^{-1}).
\]

Factoring \(R\) over \(\mathbb F_p\) yields 84 linear roots arranged into 42 reciprocal pairs \((r,r^{-1})\). Each pair contributes one root to \(P\), so there are \(2^{42}\) possible profiles before using the remaining evidence.

For a chosen root \(r\), the corresponding normalized factor is \((X-r)/(1-r)\); this makes its value at \(X=1\) equal to 1. Thus each pair offers two possible contributions to \(P(89)\). A meet-in-the-middle search over the 42 choices finds profiles matching the forward reading \(P(89)=11033196826\). There are multiple such candidates, so the forward reading alone does not identify the signed profile.

For each candidate, build the receipt message exactly as specified, including the final LF, and verify its DER ECDSA signature with the supplied P-256 public key and SHA-256. The valid candidate is the signed profile. Its coefficients, in ascending order from \(a_0\) through \(a_{42}\), are:

```text
11782335728, 15050702239, 3797047751, 7416381152, 5392461169,
11440482748, 2699918251, 1860647332, 8585001886, 15233453359,
12825134468, 10524570811, 9732538306, 5769389048, 11978964307,
12244245405, 13561421955, 11331292902, 10077157791, 3964581133,
14103796659, 11183421998, 12142137977, 13791458538, 2360805652,
7170225858, 5096156679, 5461905604, 13206392550, 472211876,
14967010902, 4413590058, 12982602456, 3366622070, 6305620312,
3842456724, 12112514060, 15080738822, 10504485684, 13156436705,
14265067822, 7957263947, 5926339596
```

Finally, XOR the decoded mask with the prefix of `SHA256(b"cold-echo-flag-pad-v1\n" + message)`, then URL-safe Base64 encode the result without padding. This gives `th3_1c3_k3pt_th3_r3c31pt`, so the flag is:

**`gctf{th3_1c3_k3pt_th3_r3c31pt}`**


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
