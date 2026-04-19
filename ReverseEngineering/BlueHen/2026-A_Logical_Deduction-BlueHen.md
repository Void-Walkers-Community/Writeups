A Logical Decuction-Writeup
On first look we see that the final gate is a AND gate, meaning every line entering the AND fate should be equal to 1
By observation, we see that a1, a9 and a11 are directly connected to the AND gate so those 2 are equal to 1
a1 , a11= 1
I divided the whole image into 9 sub parts for convinience
In the 5th and 7th and 9th subpart, we see the xor is connected to the same wire so that will result in 0 and the E layer and to get output as 1 in the D layer we need the other line, i.e the a3, a7 and a8 wire to be 1
In 6th subpart, the xor gate output at E layer will be 1, so the 6th bit will have to be 0
same with 4th subpart right side, we have same setup which gives us 10th bit as 0 since the left half of 4th subpart xor is conencted to same wire resulting in 0
from the 4th subpart we know the a10 is equal to 0 since the orgate will be giving 1 as output and the xor gate at layer D needs to give 1 as output
same logic for a2, in the 3rd subpart right most xor gate we already know or gate will result in 1 and for xor at layer D to give 1, we need a2 =0
In the 6th subpart the or fate at layer E will give 1 so for xor to give output as 1 the a5 needs to have value 0
<img width="794" height="482" alt="Napkin_1" src="https://github.com/user-attachments/assets/49f8f252-6490-4d9d-bcd6-8dc45e6a8934" />

this gives us the final string 10100101101

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
