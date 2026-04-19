Bright Night Sky-->

I first opened this challenge, I was greeted with what looked like a intercepted Soviet-era chat log written in Russian, a mysterious ciphertext at the bottom, and an image called "Bright Night Sky" the flavor text told me a dying signal officer had typed one last message ucrx{ARMBPCR GCIMF} and I needed to decode it before the soviets could launch a missile.
Dramatic. 
Before jumping into brute-forcing anything, I took a moment to actually read the conversation
The chat (translated from Russian) goes roughly like this:

You: General, are you there?
General: Yes, what is it?
You: I don't have much time, but don't forget the salt and vinegar chips — the ones I love so much. Lay's, to be precise...
General: Are you in danger?
General: You have five minutes to respond before we deploy our armed forces.

The injured officer was clearly trying to slip something past whoever might be watching. The message about chips felt way too specific to be casual small talk especially from someone who's dying and typing their last words Lay's was the clue it had to be.

The ciphertext was: ucrx{ARMBPCR GCIMF}
The flag format given was jctf{...}, which immediately told me something useful the wrapper ucrx was the encrypted version of jctf that's a known-plaintext gift. I could use those four characters to figure out exactly what cipher was being used and, more importantly, what the key was.
I mapped the transformations:

u --> j : shift of -11
c --> c : shift of 0
r --> t : shift of +2
x --> f : shift of -18

The shifts weren't consistent, which ruled out a simple caesar cipher but the pattern of shifts repeating across a keyword pointed directly to a vigenere cipher the key just had to repeat and produce those exact offsets.

finding the key:
Converting those shifts to letters (where A=0, B=1, etc.):
−11 mod 26 = 15 --> P
hmm
The challenge practically told me the key with the Lay's reference what if the key was literally LAYS?
Now I ran the full ciphertext ARMBPCR GCIMF through the vigenere cipher with key LAYS
The plaintext came out as: PROJECT ORION

Flag: jctf{PROJECT ORION}

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
