Click Here For Free Bricks solved: UMASS{TheZoo_e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae}

analysing the pcap file it shows many get requests from 10.0.0.10 to 156.233.52.16 on port 80 and there are many files passing in http so we export those files:

cooldog.jpeg -> innocent image
literallyme.jpeg -> innocent image
fungame.jpg -> innocent image
installer.py -> dropper script
launcher -> encrypted payload

the launcher binary is encrypted using libsodium SecretBox (XSalse20-Poly1305 AEAD). The seed "38093248092rsjrwedoaw3" is  SHA-256 hashed to produce the 32byte symmetric key. Then after decryption the launcher is executed and beacons out to 76.54.32.144 .

Then I replicated the dropper decryption and the decrypted binary is a FreeBSD/i386 ELF exe with 13182bytes

then I submited the hash to virustotal https://www.virustotal.com/gui/file/e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae/details and under the names field there it is TheZoo_e7a09064fc40dd4e5dd2e14aa8dad89b328ef1b1fdb3288e4ef04b0bd497ccae

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
