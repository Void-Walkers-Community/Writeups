Operation Dark Side -->
Rijndael / AES Encryption:
The challenge description explicitly mentioned "Rijndael-based encryption" — Rijndael being the original name of the algorithm that was standardized by NIST as AES (Advanced Encryption Standard). This immediately pointed to AES decryption.

Identifying the Mode:
The file provided three things: a long hex blob (ciphertext), a 32-byte hex key (256-bit), and a 16-byte IV. Having both a key and an IV together is the classic fingerprint of AES-CBC mode — CBC (Cipher Block Chaining) uses the IV to XOR against the first block before decryption.
Decrypting the Transmission:
Plugged the ciphertext, key, and IV into a standard AES-256-CBC decryption — out came a realistic-looking spacecraft telemetry log showing an attack timeline: failed SSH auth, binary injection on /bin/flight_core, privilege escalation, and finally the automated "TITAN-SHIELD" lockdown sequence triggering SAFE_MODE_V3.1.

Finding the Flag:
Buried inside the log was a suspicious NV_MEM_DUMP entry containing a base64-looking string: amN0ZntyZWxheV9zYWZlX21vZGVfYWN0aXZhdGVkXzIwMjZ9. Decoded it — and there was the flag.
Flag: jctf{relay_safe_mode_activated_2026}

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
