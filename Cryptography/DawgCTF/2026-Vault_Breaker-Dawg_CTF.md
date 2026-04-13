```
Vault Breaker — Short Writeup

We are given a PDF containing a note made of strange symbols, plus the hint:

“They told me to use a long password.... I made it longer”

The note is in the provided file.

The symbols look like a substitution cipher, possibly pigpen-style, but the easiest way to solve it is by checking the repetition pattern of the symbols instead of decoding each symbol one by one.

After comparing the 21 symbols, the repeated positions are:

1 = 5 = 7
8 = 10
11 = 19
4 = 20
16 = 17

So the ciphertext follows this pattern:

A B C D A E A F G F H I J K L M M N H D O

Now use the clue. Since Scrooge says he made a “long password” even longer, a strong candidate is:

EXTREMELYLONGPASSWORD

Check its repetition pattern:

E X T R E M E L Y L O N G P A S S W O R D

This gives the exact same structure:

A B C D A E A F G F H I J K L M M N H D O

So the plaintext matches perfectly.

Answer
EXTREMELYLONGPASSWORD
```
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
