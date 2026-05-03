# Challenge Writeup: Base
---
# Problem
We intercepted a suspicious encoded message:
```
S3ViU1RVKGI0czNfNjRfMXNfdGhlX2JhNWk1KQ==
```
---
# Analysis
First observation:
The string consists of:
* Uppercase + lowercase letters
* Numbers
* Ends with `==`
This strongly suggests **Base64 encoding**, which is a very common encoding scheme in CTFs.
---
# Approach
CyberChef was used to decode it 
---
# Output
```
KubSTU(b4s3_64_1s_the_ba5i5)
```
---
# Final Flag
```
KubSTU(b4s3_64_1s_the_ba5i5)
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
