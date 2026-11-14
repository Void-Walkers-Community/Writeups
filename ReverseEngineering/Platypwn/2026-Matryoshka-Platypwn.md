Stage 1 (the outer binary).

It is a loader with a ~154 KB encrypted blob in .data.
At startup it decrypts the blob in place with a keystream from an LCG: seed 0xed9e231, multiplier 0x7c674e63, and each key byte is the XOR of the four bytes of the state.
It then writes the result to a memfd_create file and runs it with fexecve as “stage2”.
I replicated the decryption in Python and got a valid ELF, which is stage 2.

Stage 2.

It asks for a password given as 32 hex characters, so 16 bytes.
Each 4-byte block of the password is hashed with standard CRC32, and the four results (big-endian) must equal the string my_c00l_p44sw0rd.
CRC32 over exactly 4 bytes is one-to-one, so I recovered each block by brute force over all 2³² inputs.
The recovered password is c667c58c3b02d6c91b8def4fdd1bc5f3.
The flag is the 36-byte blob at 0x1b560 XORed with the password repeated every 16 bytes.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
