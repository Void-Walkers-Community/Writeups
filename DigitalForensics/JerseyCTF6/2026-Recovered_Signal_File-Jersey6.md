Recoverd signal file -->
So I got handed a log file from a "Cold War satellite" with this blob of text:
aW9kant2ZHdob29sd2hfdmxqcWRvX2doZnJnaGd9
First thing I noticed that trailing =-less but clearly encoded string screams Base64. Threw it into a decoder and got:
iodj{vdwhoolwh_vljqdo_ghfrghg}
Okay, so something is there. The curly braces {} are a dead giveaway that the flag structure is intact, just shifted. That's a Caesar cipher. Brute-forced all 25 shifts and shift 3 gave a clean readable result:
jctf{satellite_signal_decoded}

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
