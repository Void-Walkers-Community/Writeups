Fix Your Ship-->

We're given three corrupted files and told the first one is a png the goal is to fix the files and recover the flag.

The first thing I did was check the magic bytes
file1: 89 00 00 00 0d 0a 1a 0a
file2: 00 00 00 e0 00 10 4a 46 49 46
file3: 00 00 00 20 70 66 79 74 69 73 6f 6d

Knowing file1 is a png, the correct png magic is 89 50 4E 47 0D 0A 1A 0A comparing that to what we had, bytes 1–3 were zeroed out same idea for the others a jpeg should start with FF D8 FF and an mp4 has ftyp at bytes 4–7 file3 had those four bytes scrambled to pfyt instead.

Fixing the file:
file1 (png): Patched bytes 1, 2, 3 to 50 4E 47
file2 (jpeg): Patched bytes 0, 1, 2 to FF D8 FF — but the image still wouldn't close properly because the jpeg end-of-image marker FF D9 was also stripped from the end I only figured this out from the clue in file1
file3 (mp4): swapped bytes 4–7 from pfyt back to ftyp

Once file1 opened as a PNG, it revealed a ship schematic with some interesting details hidden in plain sight. At the bottom it read:
"second file's first hex digit is FF and the last digit is D9"
That's what tipped me off that the JPEG was missing its EOI marker

Extracting the flag:
Each file held one piece of the flag, tucked in the top-right corner or displayed on screen:
file1 (PNG schematic) --> jctf{starship
file2 (JPEG schematic) --> _voyager_
file3 (MP4 video) --> apollo}

Flag: jctf{starship_voyager_apollo}
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
