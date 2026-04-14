I first looked through the PNG structure and saw some strange custom chunks between the normal IDAT chunks. That was the real clue. Extracting them gave a hidden JPEG. Since the hint said alpha = 0.45 I tried the BPCS route. The output was still encoded and decoding it with base91 gave the final flag: UMASS{0N3_D4Y_Y0U_W1LL_83_3MPL0Y3D}
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
