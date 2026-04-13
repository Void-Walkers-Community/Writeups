Through the Looking Bit solved: flag DawgCTF{R3ync_1s_b3tt3r_th5n_http}

So I assumed that the university that is being refered to in the challenge is the UMBC university

by browsing I found the url for the mirrors of the uni mirror.lug.umbc.edu and ran the rsync command to look at the banner rsync rsync://mirror.lug.umbc.edu

![image](https://github.com/user-attachments/assets/3cc71a0d-64c7-4f7c-9ca5-5f41481e9d1b)

after looking at the banner I decided to decode the binary text
it just gave some giberish text, so I re read the chall description and then it clicked me that this might be inverted so maybe a bitwise NOT operation (turn all 0 to 1 and all 1 to 0) would be the play. And it was ::D

then I converted everything to decimal and from decimal to ascii and then got the flag ::D

* [🔙 Back to Miscallaneous Directory](../Miscellaneous)
* [🔙 Back to Miscellaneous Index Directory](../Miscellaneous/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
