## We have an incident!4

Category: Forensics

### Summary

The compromise started on the HR workstation when `WINWORD.EXE` opened `C:\Users\Elvira\Downloads\Ð ÐµÐ·ÑŽÐ¼Ðµ.docm` on `2026-03-28 13:20:43Z`. That document spawned `cmd.exe`, which launched hidden PowerShell to fetch `runme.txt`, then `implant.ps1` from `192.168.100.54`.

From the HR PowerShell and Sysmon timeline, the attacker executed these payloads in order:

1. `Ð ÐµÐ·ÑŽÐ¼Ðµ.docm`
2. `Certify.exe`
3. `Rubeus.exe`
4. `mimikatz.exe`
5. `wlmss.exe`

### Privilege Escalation

`Certify.exe` requested a certificate from the `VulnerableUserSAN` template with `AltName : admin`, and `Rubeus.exe` then used PKINIT to obtain a TGT for `kuban.loc\\admin`. The exploitable condition is an AD CS `ESC1` misconfiguration: an enrollee-supplied subject/SAN template usable for authentication.

### Exfiltration

Two items were directly evidenced as transferred:

1. `0-40e10000-admin@krbtgt~kuban.loc-KUBAN.LOC.kirbi`
   - HR classic PowerShell log shows a TCP send to `192.168.100.54:9000`.
2. `ntds.dit`
   - AD PowerShell/Application/Sysmon show `ntdsutil` IFM creation, copy to `C:\Users\Public\ntds.dit`, then a TCP send to `192.168.100.54:9001`.

### Flag

KubSTU{ESC1:Резюме.docm_Certify.exe_Rubeus.exe_mimikatz.exe_wlmss.exe:0-40e10000-admin@krbtgt~kuban.loc-KUBAN.LOC.kirbi_ntds.dit}
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

