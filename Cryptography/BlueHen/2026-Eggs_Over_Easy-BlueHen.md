Eggs Over Easy - Writeup
If you know what goes well with eggs, this will be over easily...

We had a file with plain text and after running  'file txt' and got this output:
txt: ASCII text, with very long lines (464), with no line terminators

I kept the text in a invisible decoder and found it was a combinations of space and tabs and converted them into A and B using the command cat txt | tr ' \t' 'AB' thinking i have to use Baconian cipher because that was what i got on my google search for ciphers related to eggs and bacon
That gave nothing useful so i converted them into 0s and 1s and used the Binary Decoder and got the following string
ff35 ff24 ff23 ff34 46 ff5b ff22 ff14 ff43 ff4l ff4j ff5d 

A mquick google search told me about fullwidth characters which are characters shifted by 0XFEE0 so we simply subtract that from each of the values we have
Script:
h = "ff35 ff24 ff23 ff34 46 ff5b ff22 ff14 ff43 ff4f ff4e ff5d"
flag = ""
for x in h.split():
    v = int(x, 16)
    flag += chr(v - 0xfee0 if v > 0xff00 else v)
print(flag)

This gave the flag
UDCTF{B4con}
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
