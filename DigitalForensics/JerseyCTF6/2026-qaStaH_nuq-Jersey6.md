qaStaH nuq---> 
I opened the file with tshark got 366 packets total. Mix of SSH, TCP data and DNS, and only one HTTP packet and 50 ICMP packets. That lone HTTP packet immediately stood out to me everything else is either encrypted (SSH) or bulk noise.
Anlyzed the single HTTP packet it had a base64 string in it : amN0ZntBdHRhY2tfVGhlX0VudGVycHJpc2V9

decoded it and got the flag : jctf{Attack_The_Enterprise}

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
