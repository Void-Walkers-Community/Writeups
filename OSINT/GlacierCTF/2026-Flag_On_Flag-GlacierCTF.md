Flag on the flag — OSINT
The challenge asked us to recover information hidden on the LosFuzzys team banner.

I searched public team photographs from on-site events. An Insomni’hack post initially looked promising, but the banner was not sufficiently readable there.

A LosFuzzys LinkedIn post about Sardinia CyberCamp contained a clearer view. The third photograph showed a hexadecimal string along the banner’s lower edge.
Extracting the message
I inspected an enlarged, perspective-corrected view of the banner and transcribed the hexadecimal characters.

Decoding the hexadecimal produced bytes that were not immediately readable. Using the expected flag prefix as known plaintext, I tested whether the data had been masked with a single-byte XOR key:
[
k=C_0\oplus P_0
]
The first byte suggested 0x42. Checking the remaining prefix bytes independently produced the same key.

I then XORed every decoded byte with 0x42. The resulting text matched the expected format.
Verification
The XOR key was supported by every byte of the known prefix, rather than just one guessed character. The complete decoded message was coherent and correctly delimited.

This recovered the banner’s hidden message; challenge-server acceptance was not checked. 

---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
