Play Fair, Punch! writeup

1. Recognize the image

The PNG is an old IBM-style punch card. The black rectangles are the actual punches.
The printed "FORTRAN STATEMENT" text overlaps some of the upper rows, so OCR is not reliable.
The right way to read it is column by column using punch-card / Hollerith encoding.

2. Decode the punch card

Using the standard alphabet mapping:

12 + 1..9 -> A..I
11 + 1..9 -> J..R
0  + 2..9 -> S..Z

Reading the 34 punched columns gives:

OEGFDKGKRYRYOELTAGELGFPEWBFLRPLDCY

3. Use the title hint

The challenge title is "Play Fair, Punch!", which strongly hints:

- Playfair cipher
- key = PUNCH

Build the 5x5 Playfair square with I/J merged:

P U N C H
A B D E F
G I K L M
O Q R S T
V W X Y Z

4. Decrypt

Decrypting the ciphertext with Playfair and key PUNCH gives:

SAMANDMISXSXSAMSPACEMACAQUEMONKEYS

5. Remove Playfair filler X's

The substring MISXSXSAM is the Playfair-expanded form of MISSSAM.
So after removing filler X characters inserted between doubled letters, the plaintext reads:

SAMANDMISSSAMSPACEMACAQUEMONKEYS

Readable form:

SAM AND MISS SAM SPACE MACAQUE MONKEYS

6. Interpretation

The recovered clue points to Sam and Miss Sam, the famous rhesus space monkeys.
That is the intended lineage clue hidden in the card.

Recovered plaintext clue:

SAMANDMISSSAMSPACEMACAQUEMONKEYS

Human-readable answer:

Sam and Miss Sam

Likely flag body if the challenge wants the raw decrypted text:

SAMANDMISSSAMSPACEMACAQUEMONKEYS
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
