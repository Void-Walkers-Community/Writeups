# cat_girl Challenge Writeup

The challenge provided a hex string in 'what_could_this_mean.txt' and a directory structure containing 39 folders named '0-9', 'A-Z', '_', '{', '}'. Each folder contained images of 'cat girls'.

## Analysis
1. The hex string was 3968 characters long. Each SHA-256 hash is 64 characters.
2. 3968 / 64 = 62. So the string contains 62 character hashes.
3. Each character of the flag corresponds to an image in the folder named after that character.
4. By hashing all images in the subdirectories and comparing them against the chunks of the hex string, the flag was reconstructed.

## Solution Script
A Python script was used to walk through the directories, compute SHA-256 hashes for all .jpg files, and map them to their respective characters.

## Flag
KUBSTU{A7_LE4ST_N0W_Y0U_H4V3_A_BUNCH_0F_P1CTUR3S_OF_C4T_GIRL5}


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
