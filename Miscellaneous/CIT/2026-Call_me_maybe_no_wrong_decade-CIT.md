# Call me, maybe? No... wrong decade.

## Challenge

**Category:** Crypto / Password Cracking  
**Flag format:** `CIT{password}`

The challenge gave a bcrypt hash:

```text
$2b$10$ni0U3D1bg1NY6G/k8CDhuXG7m/WNZzuV/9PDpNrgKs4wUjaTwGO
```

The title was the main hint:

```text
Call me, maybe? No... wrong decade.
```

## Solution

The string starts with `$2b$10$`, which identifies it as a **bcrypt** password hash.

```text
$2b$10$...
```

The title references the song **Call Me Maybe**, but says **wrong decade**.  
Instead of the 2010s song, this points to an older famous “call me” reference from the 1980s:

```text
867-5309/Jenny
```

This is a song by Tommy Tutone released in the early 1980s. Since the challenge asks for a password, the likely candidate is a normalized version of the phone number and name:

```text
8675309jenny
```

## Verification

We can verify the candidate against the bcrypt hash using Python.

```python
import bcrypt

target = b"$2b$10$ni0U3D1bg1NY6G/k8CDhuXG7m/WNZzuV/9PDpNrgKs4wUjaTwGO"
candidate = b"8675309jenny"

if bcrypt.checkpw(candidate, target):
    print("Password found:", candidate.decode())
```

Output:

```text
Password found: 8675309jenny
```

## Flag

```text
CIT{8675309jenny}
```

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

