Broken Signal-->

We're given two files: a wav audio recording and a damaged reference sheet.

The reference sheet is a 6x5 grid of characters (A to Z plus {, }, ., _), with two numbers visible on it: 252 and 1530 large portions of the sheet are described as missing or unreadable.

I threw the audio into a Morse code decoder first
Output:
E E#E#AT#EI EU #SW##NSIWIE

Garbled

then I loaded the audio into Python and ran a spectrogram that's when things got interesting instead of the irregular dots and dashes you'd expect from morse, I was seeing pairs of simultaneous tones repeating in clean, equally-spaced bursts of about 0.74 seconds each.

The tones fell into two groups:

Low frequencies: 250, 310, 520, 750, 990 Hz
High frequencies: 1530, 2980, 3130, 5620, 6000 Hz

Two tones playing at once, switching in sequence this is a dual-tone encoding system, like a more exotic version of DTMF

Cracking the grid:
Going back to the reference sheet a 6x5 grid with 252 and 1530 annotated on it five low frequencies, five high frequencies, five columns in the grid it clicked:

Low frequency = selects the row
High frequency = selects the column

then mapped it out

Decoding:
Running through all the detected tone pairs in order:
(310, 6000) --> J
(250, 3130) --> C
(750, 6000) --> T
(310, 1530) --> F
and so on, the full sequence decoded to:
JCTFVLOSTYINYOSMICYSTATICW

JCTF right at the front that's the flag prefix the rest resolves to LOST_IN_COSMIC_STATIC

Flag: jctf{lost_in_cosmic_static}

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
