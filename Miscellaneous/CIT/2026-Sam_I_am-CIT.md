# SAM, I am — CTF Writeup

## Challenge

The challenge provided a hash from a dumped Windows SAM database:

```text
97a3e51e5a006eccac91e0ceabd4965b
```

The description also stated that the password policy required:

```text
5 characters + complexity
```

Flag format:

```text
CIT{password}
```

## Initial Analysis

A Windows SAM dump usually contains NTLM password hashes.  
The given value is 32 hexadecimal characters, which matches the length of an NTLM hash.

So the target hash was treated as:

```text
Hash type: NTLM
Hashcat mode: 1000
```

Save the hash into a file:

```bash
echo '97a3e51e5a006eccac91e0ceabd4965b' > hash.txt
```

## Failed Attempts

Some targeted masks were tested first, such as:

```bash
hashcat -m 1000 hash.txt -a 3 '?u?l?l?l?l?d'
hashcat -m 1000 hash.txt -a 3 '?u?l?l?l?l?s'
hashcat -m 1000 hash.txt -a 3 '?u?l?l?l?d?s'
hashcat -m 1000 hash.txt -a 3 '?u?l?l?l?l?l?d'
hashcat -m 1000 hash.txt -a 3 '?u?l?l?l?l?l?s'
```

However, all of these returned:

```text
Status: Exhausted
Recovered: 0/1
```

This meant the password did not match those specific character layouts.

## Correct Approach

The hint said the password policy was `5 characters + complexity`.

Instead of guessing a specific structure, the full printable ASCII keyspace for exactly 5 characters was brute-forced with Hashcat:

```bash
hashcat -m 1000 -a 3 -O -w 3 hash.txt '?a?a?a?a?a'
```

Explanation of options:

| Option | Meaning |
|---|---|
| `-m 1000` | NTLM hash mode |
| `-a 3` | Mask attack |
| `-O` | Optimized kernel |
| `-w 3` | Higher workload profile |
| `?a` | All printable ASCII characters |

The full mask:

```text
?a?a?a?a?a
```

means Hashcat tries every possible 5-character printable ASCII password.

## Crack Result

Hashcat successfully cracked the hash:

```text
97a3e51e5a006eccac91e0ceabd4965b:C1t!!
```

The plaintext password was:

```text
C1t!!
```

This matches the policy:

| Requirement | Satisfied By |
|---|---|
| 5 characters | `C1t!!` |
| Uppercase | `C` |
| Lowercase | `t` |
| Digit | `1` |
| Special character | `!!` |

## Flag

Using the required flag format:

```text
CIT{C1t!!}
```

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

