<img width="1192" height="805" alt="Pasted image 20260419192750" src="https://github.com/user-attachments/assets/c65b5ecb-b7ac-4f3c-8611-cdbd25a84f92" />

The challenge file is actually a `User Profile Folder` from a windows machine.

<img width="2121" height="103" alt="Pasted image 20260419192939" src="https://github.com/user-attachments/assets/08ded6e2-e2e2-48bd-892d-8f6cf63aed1d" />

This challenge asks us the date and time at which a particular website was accessed and it also prompted the user to run a powershell command.

We need the history of websites accessed using a browser like Chrome/Edge:-

The path for History of Edge is as follows:-

```
ayush-parab@Ubuntu-VM:~/ayush/CTF/CITCTF2026/click_may_fixed/kurt_backup/AppData/Local/Microsoft/Edge/User Data/Default$ file History
History: SQLite 3.x database, last written using SQLite version 3035005, file counter 2, database pages 31, cookie 0x18, schema 4, UTF-8, version-valid-for 2
```

The file is a `SQLite 3.X` database. So we open it using `DB browser for SQLite` and run the following query to get the timestamps in the format specified.

<img width="2041" height="926" alt="Pasted image 20260419125221" src="https://github.com/user-attachments/assets/4d9c2c3f-dcef-45ee-8060-6c52b281a7a3" />

In entry number 2 of above output, the user searched about `Free RAM` in `bing` after which he clicked on a malicious site which has no domain name. Surely this must be the one we are looking for.

Using its timestamp, we get the flag!

Flag - `CIT{2026-04-18T07:07:26Z}`


---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
