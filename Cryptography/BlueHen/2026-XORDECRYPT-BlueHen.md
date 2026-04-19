We were given a noisy image file named 0x67.png and the prompt "I feel CHAINED to my desk, looking for some positive FEEDBACK."
The hint pointed to Cipher Block Chaining mode and the filename 0x67 was the Initialization Vector. Since there was no encryption algorithm or key provided, the image was simply scrambled using basic mathematical chaining: Plaintext Byte = Current Ciphertext Byte XOR Previous Ciphertext Byte.
Upon flattening all RGB layers into one continuous list of bytes, each byte was simply XORed with the one right before it gave a dark image and that contained the flag
UDCTF{xOr_TO_Th3_FLa@g}
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
