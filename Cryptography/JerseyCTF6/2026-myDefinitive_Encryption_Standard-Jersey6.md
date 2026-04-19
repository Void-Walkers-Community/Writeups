myDefinitive Encryption Standard-->

The challenge drops a Python file and a ciphertext, with the description: "We have been asked to create a similar encryption standard to DES to work on lightweight IBM computer systems." The name myDefinitive Encryption Standard is already a wink. mDES.
Opening the file, you see three functions: aaa, bbb, and ccc, plus an encrypt wrapper. The naming is deliberately unhelpful, but the structure isn't that hard to read once you sit with it.
There's also this at the bottom:
key_hint = 0xD4D4A1A0  #     
ciphertext = b"9\xbd/\x9588\x0bwo\xce+\xd4*\xd8\xda\x8d\x1f*\xac\x07f\xf1a\x9b\xd7$O\xbdU\\\xe2\xc5"

understanding the ciphertext:
Let's rename the functions to something human-readable and trace through what's actually happening

Key Schedule (aaa)

def aaa(k):
    ks = []
    for i in range(ROUNDS):
        k = rot(k, 3)                                    # rotate left 3 bits
        ks.append((k ^ (0x9E3779B9 * (i + 1))) & 0xffffffff)
    return ks

This generates 4 round keys. Each iteration rotates the key left by 3 bits and XORs with a multiple of 0x9E3779B9 the golden ratio constant, the same one used in TEA, XTEA, and plenty of other lightweight ciphers, nice touch.

Round Function (bbb)

def bbb(r, k):
    return (rot(r ^ k, 5) * 0x45D9F3B) & 0xffffffff

XOR with the round key, rotate left 5, then multiply by 0x45D9F3B. That multiply is doing the work of diffusion here it's a common trick in lightweight cipher design to approximate an S-box without the lookup table overhead.

Feistel Network (ccc)

def ccc(block, keys):
    l = int.from_bytes(block[:4], "big")
    r = int.from_bytes(block[4:], "big")
    for k in keys:
        l, r = r, l ^ bbb(r, k)
    return r.to_bytes(4, "big") + l.to_bytes(4, "big")

Classic Feistel structure split the 8-byte block into two 32-bit halves, run 4 rounds, output with a final swap the comment in the original code says "# final swap back" because in a standard Feistel you'd swap after the last round, and this swaps back to undo it. Or equivalently, outputting r || l instead of l || r achieves the same thing.
The whole cipher is a Feistel network without an S-box which is literally what the flag says.

Decryption:

To reverse a Feistel round, you use the fact that each round is self-contained:
Forward:  l_new = r_old
          r_new = l_old XOR bbb(r_old, k)

Reverse:  r_old = l_new
          l_old = r_new XOR bbb(l_new, k)

The round function bbb doesn't need to be inverted you just apply it to the half you already know that's the elegant property of feistel ciphers.
The only wrinkle is inverting the multiply. bbb multiplies by 0x45D9F3B mod 2³² to undo that, we need the modular inverse:

inv_mult = modinv(0x45D9F3B, 2**32)  # = 0x119DE1F3

so the full decrypt:

def decrypt_block(block, keys):
    rN = int.from_bytes(block[:4], "big")
    lN = int.from_bytes(block[4:], "big")
    l, r = lN, rN                          # undo the final output swap
    for k in reversed(keys):
        l, r = r ^ bbb(l, k), l           # reverse each round
    return l.to_bytes(4, "big") + r.to_bytes(4, "big")

Finding the key:

The hint is 0xD4D4A1A0 the challenge says a letter or two might be missing the key is a 32-bit integer, so "missing a letter" most naturally means a hex digit is wrong.
Rather than brute-forcing all 4 billion possibilities, we narrow it down: try replacing each of the 8 hex nibbles one at a time, then in pairs look for output starting with jctf{.
Block 0 of the ciphertext decrypts to jctf{f3i with key 0xD4D4A1A5. That's a single nibble off from the hint (0 --> 5 in the last position). Running the full decrypt:

jctf{f3ist&l_fun_w1thout_sb0\x00\x00x}

30 of 32 bytes are printable ASCII the two \x00 bytes are zero-padding to reach the 32-byte block boundary, sitting just before the closing x}.

Flag: jctf{f3ist&l_fun_w1thout_sb0x}
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
