I got 2 segment for challenge "Cold Wake"
first one was found in Tape 1 exif metadata:
Software field: GRAVITY-WEAPON OS v1.3
Artist field: Cold War Orbital Research Division
Date: 1982:06:12 09:14:00 (Cold War era, deliberately retro)
JPEG Comment: ORBITAL LAB ARCHIVE :: SINGULARITY INIT SEGMENT :: SEQ=47291

That SEQ=47291 is the payload. Segment 1 = 47291

second segment found in tape 2 afte running stegseek on it found a hidden file embedded in the JPEG pixel data, protected by the password galaxy inside was a file called Tape2Paper_small.jpg  a photo of an old piece of paper with a number stamped on it in large stencil font:
80536

therefore segment 2 =80536

for third segment i again ran stegseek on tape 3 with the same password galaxy, but this time the hidden file was an MP3 audio file called Tape3_small.mp3 just 4 seconds long
this is the last segment for the flag but i am not able to analyze it i opened the audio file using audacity but the text is unclear

<img width="688" height="299" alt="image" src="https://github.com/user-attachments/assets/03d4923d-24f9-439d-a388-5a500f754b1e" />

Flag: JCTF{47291-80536-19408}

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
