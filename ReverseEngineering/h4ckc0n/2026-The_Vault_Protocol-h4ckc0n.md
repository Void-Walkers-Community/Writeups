Writeup: vault
Recon. vault is a stripped, statically linked C++ x86-64 ELF. It prompts Enter the flag:, then prints Wrong. or Correct!. The main code is padded with decoy arithmetic loops, but the real logic is short.

The check.
The input must be exactly 25 bytes (cmp [rsp+0xc8], 0x19).
Each byte is XORed with a 5-byte repeating key, 5a c3 17 9e 2b. This is the "hidden key".
Six transforms run on the buffer:
Rotate byte i left by (i % 7) + 1.
Prefix sum mod 256.
Permute positions: byte n goes to (3 + 7n) mod 25.
Substitute through a 256-byte S-box. It is built lazily at runtime into 0x50e7e0, so I dumped it with gdb.
A chained step, b'[j] = rol(b'[j+1], 3) ^ b[j], running from the end of the buffer backward.
An XOR with the stream s = 5s + 0x11, starting at 0x54.
The result is compared with 25 constant bytes: 16 at 0x4d8a80, 8 at 0x4d8bd8, and a final 0x62.

Solve. Every stage is a bijection, so there is no brute force and exactly one valid input. I dumped the S-box, key and constants from a gdb breakpoint at the compare. Then I inverted the stages in reverse order: the XOR stream, the chained rotate, the inverse S-box, the inverse permutation, the un-prefix-sum, the right-rotate, and finally the key XOR. My forward model of the pipeline reproduces the target exactly, and the real binary prints Correct! for the recovered string.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
