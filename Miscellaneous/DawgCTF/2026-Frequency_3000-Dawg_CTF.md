Flag: DawgCTF{WHYN0TZO!DB3RG?}

Step 1 – Hex decode
flag.txt decimals → ASCII:
DawgCTF{ 390 1002 580 1314 191 1589 33 1526 141 762 352 88 1293 379 50 }

Step 2 – Transcript frequency
Hint says use transcript of Futurama pilot “Space Pilot 3000”.
“Frequency 3000” means character frequency in that transcript.

Count frequency of every char (letters, digits, punctuation, spaces) in transcript. Sort most→least frequent. This is the “frequency alphabet.”

Map each number (1-indexed) to that alphabet:

390 → w
1002 → h
580 → y
1314 → n
191 → 0
1589 → t
33 → z
1526 → o
141 → !
762 → d
352 → b
88 → 3
1293 → r
379 → g
50 → ?

Result: WHYN0TZO!DB3RG?

Step 3 – Leetspeak
WHYN0TZO!DB3RG? reads as “Why not Zoidberg?” – famous Futurama line.



---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

Final flag: DawgCTF{WHYN0TZO!DB3RG?}
