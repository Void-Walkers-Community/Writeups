Return by d34th
The challenge gave me parcel_delivery.exe and said it needed a tracking ID. I inspected the executable’s readable strings and found:
woah_a_rezero_mystery
The program accepted this as the tracking ID and asked for a recovery code.

I then examined the executable’s validation functions. The code had to be 16 digits long, using only 1 through 4. The program treated those digits as a 4×4 grid and checked that every row and column contained each digit once.

It also stored four grid clues in an encoded form. The decoding key was 40, obtained from the number of Re:Zero facts in the program. After decoding the clues and solving the grid, I got:
4 3 1 2
3 2 4 1
1 4 2 3
2 1 3 4
Reading the rows from left to right gave the recovery code:
4312324114232134
I entered the tracking ID and recovery code into the program. It accepted the grid and used the code to decrypt and print the flag:
d4rk{r3turn_0f_r3m_40} 

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
