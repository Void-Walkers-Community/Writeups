## badyuri Writeup

We are given two text files:

* `yuri.txt`
* `yuri_1.txt`

Challenge hint:

> “Corporate wants you to find the difference between these two files. They are not the same file.”

That strongly suggests a diff-based challenge.

### Step 1: Compare the files

Running a diff shows that most of the files are identical, but scattered throughout `yuri_1.txt` there are single-character corruptions.

Examples:

```text
had  -> haÆ
her  -> hÈr
stood -> sèood
eldritch -> eÒdritch
Additionally -> ¼dditionally
```

Each corruption changes exactly one character.

---

### Step 2: Check ASCII values

The weird characters looked intentional, so I checked the byte/ASCII values of the original vs modified characters.

Example:

```text
d = 100
Æ = 198

198 - 100 = 98
98 = 'b'
```

Another:

```text
e = 101
È = 200

200 - 101 = 99
99 = 'c'
```

Another:

```text
t = 116
è = 232

232 - 116 = 116
116 = 't'
```

That gives:

```text
bct...
```

Clearly the flag is being encoded as:

```text
modified_char - original_char = flag_character
```

---

### Step 3: Extract all modified characters

There are 30 altered characters total.

For each:

```python
flag += chr(ord(modified) - ord(original))
```

Script:

```python
orig = open("yuri.txt","rb").read()
mod  = open("yuri_1.txt","rb").read()

flag = ""

for a,b in zip(orig,mod):
    if a != b:
        flag += chr(b-a)

print(flag)
```

---

### Step 4: Recover flag

Running it gives:

```text
bctf{w3_l0ve_yur1_rB4DN8aULH9}
```

## Flag

```text
bctf{w3_l0ve_yur1_rB4DN8aULH9}
```

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
