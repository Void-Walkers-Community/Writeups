Writeup
Initial Analysis
The challenge provided a file named logged_convo.txt, containing six messages exchanged between two people, Brew and Drew.

Instead of normal text, every message was represented by an extremely large decimal number. The messages alternated between Brew and Drew, suggesting that these numbers contained an encoded conversation.

At first, the values resembled large integers used in cryptographic algorithms such as RSA. However, the file contained no public keys, modulus, or other information normally associated with RSA encryption.

This suggested that the numbers might represent data in a different format rather than conventional encrypted messages.
Finding the Pattern
The first step was to examine the mathematical properties of the numbers.

An interesting observation was that all six numbers were divisible by 17.

After dividing the numbers by 17 and examining their binary representations, a consistent structure became visible.

The resulting binary values could be arranged into groups of 17 bits. These groups contained patterns of consecutive zeros and ones that appeared to represent pixel data.

This was the main breakthrough: the numbers were hiding images of text.
Reconstructing the Conversation
To make the hidden information readable, the following approach was used:
1. Divide each decimal number by 17.
2. Convert the result into binary.
3. Pad the binary representation to 1,751 bits, giving 103 groups of 17 bits.
4. Treat each 17-bit group as a vertical column of pixels.
5. Arrange the columns from left to right, reversing the vertical bit order to display the characters correctly.
In this representation, a binary 1 corresponds to a filled pixel, while a binary 0 represents an empty pixel.

Once the pixels were arranged correctly, the seemingly random numbers transformed into readable text.

Decoding the Messages
The six values revealed the conversation between Brew and Drew.
Sender	Recovered message
Brew	Hi! How ARE YOU?
Drew	I AM SCARED ☹
Brew	WHY???
Drew	Is This SAFE?
Brew	d4rk{0bV_DUHHH?}
Drew	OK! Thanks (smiley)

Flag: d4rk{0bV_DUHHH?} 

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
