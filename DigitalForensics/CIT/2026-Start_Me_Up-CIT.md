# Start Me Up - Last challenge in this chain

<img width="1194" height="567" alt="Pasted image 20260419194448" src="https://github.com/user-attachments/assets/1adab14d-871e-4679-bc03-44753a58ed5c" />

We have to check for persistence! The title suggests that the malicious script `DiskCleaner.ps1` we saw in the previous challenge has been set to run automatically on startup for persistence.

We will check the startup folder at:-

`/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup`

We notice a text file there with the following output:-

```
ayush-parab@Ubuntu-VM:~/ayush/CTF/CITCTF2026/click_may_fixed/kurt_backup/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup$ cat e9fje2.txt 
Q0lUe3N0NHJ0X20zX3VwX2kxMV9uM3Yzcl9zdDBwfQ==
```

This appears to be a `base64` string, decoding it gives us our flag!

Flag - `CIT{st4rt_m3_up_i11_n3v3r_st0p}`

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
