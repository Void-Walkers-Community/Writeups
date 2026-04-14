The Accursed Lego Bin solved: UMASS{tH4Nk5_f0R_uN5CR4m8L1nG_mY_M3554g3}

so the encoder shows that it encrypts "I_LOVE_RNG" with RSA with e=7 and random 2048bit primes, creating n and a cipher "seed" then it convertsd the flag into a flat array of bits and then it shuffles those bits 10 timmes, everytime  seeding through python random with seed * (i+1) for i in 0..9. Theb ut writes the shuffled flag and enc_seed = pow(seed, e, n) to output.txt

the thing is, n is never written to the output, but as you can see there is a flaw in the math here. 
so the RSA is defined as: ciphertext = plaintext^e mod n
the mod n step is what makes this hard to reverse, but mod n only does anything when the plaintext^e actually exceeds n.

so:

"I_LOVE_RNG" is 10 bytes = 80 bits of plaintext
e=7, so plaintext^7 is at most 560bits
n is the product of two 2048bit primes, making it ~4096bits

560bits << 4096 bits. So the plaintext^7 never wraps around modulo n.

that means: seed = plaintext^7

so we can compute seed without knowing n. we know the plaintext "I_LOVE_RNG" and we know e=7, so:
m = int.from_bytes(b"I_LOVE_RNG", "big")
seed = m ** 7

the same thing applies a second time. seed ~=548b, so seed^7 ~= 3832bits, it is less than the 4096bit modulus, that means enc_seed=seed^7 exactly and we can verify our recovered seed against the outpupt file to confim

so once we have the seed, undoing is easy. the encoder did 10 shuffles in order:
for i in range(10):
    random.seed(seed * (i + 1))
    random.shuffle(flag_bits)

to reverse, we do the reverse order (i=9 down to 0) and invert each ppermutation:
then after 10 inversions, we have the original bit array, which we decode to bytes and then to ASCII, and then we get the flag



---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
