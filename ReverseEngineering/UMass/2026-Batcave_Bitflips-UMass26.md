Batcave Bitflips solved: UMASS{__p4tche5_0n_p4tche$__#}

so the program is a 64-byte state hash that takes a 32char key. it calls expand_state which fills the 64-byte state as state[i] = key[i % 32]^i (XORing each byte with its own index), then runs 0xBEEEEF rounds of three operations: 
substitute (SBOX lookup on all 64bytes)
mix (in-place XOR: state[i] ^= state[(i+1)%64] ^ state[63-i])
and rotate

then derive_final folds the 64byte state in half with XOR to do a 32byte hash, then hash is compared to EXPECTED in .data and if it matches decrypt_flag it XORs the hash against the encrypted FLAG bytes.

for this chall I used the provided hints in the platform and the hints say 3 bugs, all findable from static analysis:

Bug 1 (rotate "Rotation rotation rotation!"): the rotate function is supposed to do ROL8 by 3 on each byte ((b << 3) | (b >> 5)) & 0xFF. the disassembly at 0x1280 has shr $0x6,%al instead of shr $0x5,%al so only the top 2 bits fold back instead of the top 3, doing a garbled non-permutation instead of a clean rotation.

Bug 2 (SBOX "Something about that SBOX seems off..."): the 256byte SBOX at 0x4080 is not a valid permutation. value 0x43 appears two times at indices 0x18 and 0x5c and value 0x44 is missing, so the bug is at index 0x18: it should be 0x44.
this matters because a non-bijective SBOX is irreversible and does the wrong hash entirely.

Bug 3 (decrypt_flag): the decrypt_flag function at 0x12ec has or %eax,%ecx (opcode 09 c1) but it should be xor %eax,%ecx (opcode 31 c1) to properly XOR the encrypted FLAG bytes against the hash key. OR'ing them just scrambles the output.

then the password !_batman-robin-alfred_((67||67)) is sitting in plaintext in .data at 0x4020, right before EXPECTED at 0x4040.

so just patch the byte 0x1282 from 06 to 05, patch SBOX[0x18] from 0x43 to 0x44, patch byte 0x12ec from 09 to 31, feed the 32byte password in and the fixed hash matches EXPECTED, decrypt_flag XORs correctly and the flag prints right out.



---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)



