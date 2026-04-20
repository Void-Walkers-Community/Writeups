# CTF Writeup — Blind Hens (100 pts) | Steganography

**Category:** Steganography  
**Difficulty:** Medium-Hard  
**Solves:** 133  
**Flag:** `UDCTF{0m6_wh173_5p4c3_d3c0d1n6_15_4_7h1n6?}`

---

## Challenge Description

> We recovered a routine internal memo from a workstation flagged during an investigation. Nothing in the visible contents appears unusual, but analysts believe a message was hidden in plain sight. Review everything carefully — even the smallest details can be important. Some internal policies may hold the key.

We got one file: `memo.txt`

---

## First Look

I opened the file. It looked like a normal office memo — badge access, conference rooms, printer maintenance. Very boring stuff.

But two things looked strange to me:

**1. The hint "policy #23"**

The memo says: *"Oh btw please ensure compliance with policy #23"*

Then it lists 147 policy notes. That is a lot. Why?

**2. The lines stopped at 99 but then continued with different spacing**

Policy notes 1–99 had no trailing whitespace. But from line 100 onward, every line had something after the text. I could not see it in a normal text editor, but I felt something was there.

---

## Finding the Hidden Data

I opened the file in `xxd` (hex dump tool) to see the raw bytes:

```bash
xxd memo.txt | head -60
```

I saw something interesting — after some lines, there were `09` bytes (tab = `\t`) and `20` bytes (space = ` `) before the newline `0a`. These are **invisible characters**.

This is when I remembered a technique called **whitespace steganography**. I read about it on a CTF blog before. The idea is simple: tabs and spaces at the end of lines look like nothing, but they can carry binary data.

> **How I knew it was whitespace steganography:**  
> The challenge name is "Blind Hens" — you cannot *see* the hidden data. Also the description says "hidden in plain sight" and "smallest details." These are classic hints for whitespace stego. I also searched online and found a tool called **SNOW** (Steganographic Nature Of Whitespace) which does exactly this.

---

## Step 1 — Extract the Trailing Whitespace

I wrote a Python script to pull out all the trailing tabs and spaces from each line:

```python
with open('memo.txt', 'r') as f:
    lines = f.readlines()

for i, line in enumerate(lines):
    stripped = line.rstrip('\n')
    trail = ''
    for c in reversed(stripped):
        if c in (' ', '\t'):
            trail = c + trail
        else:
            break
    if trail:
        print(f"Line {i+1}: {repr(trail)}")
```

Output (first few lines):

```
Line 5:  ' \t\t  \t  '
Line 6:  ' \t\t \t\t \t'
Line 7:  ' \t\t  \t  '
Line 8:  ' \t\t  \t\t\t'
...
```

There were **tabs and spaces hidden at the end of lines**. This confirmed my guess.

---

## Step 2 — Convert to Binary

The classic whitespace stego encoding:
- `TAB (\t)` = **1**
- `SPACE ( )` = **0**

I updated my script:

```python
bits = []
for line in lines:
    stripped = line.rstrip('\n')
    trail = ''
    for c in reversed(stripped):
        if c in (' ', '\t'):
            trail = c + trail
        else:
            break
    for c in trail:
        bits.append('1' if c == '\t' else '0')

print(''.join(bits))
```

Output:

```
0110010001101101011001000110011101100100001100100101011001011001...
```

Total: **478 bits**

---

## Step 3 — Decode the Bits

I grouped the bits into 8-bit chunks and decoded as ASCII:

```python
bit_str = ''.join(bits)
chars = []
for i in range(0, len(bit_str) - 7, 8):
    byte = bit_str[i:i+8]
    val = int(byte, 2)
    chars.append(chr(val) if 32 <= val <= 126 else f'[{val}]')

print(''.join(chars))
```

Output:

```
dmdgd2VYE04VfFRLEhQQfBZTF0AQfEcQQBNHEk0VfBIWfBd8FEsSTRUcXg=
```

That looks like **Base64!** The `=` at the end is a clear sign.

---

## Step 4 — Decode Base64

```python
import base64
decoded = base64.b64decode('dmdgd2VYE04VfFRLEhQQfBZTF0AQfEcQQBNHEk0VfBIWfBd8FEsSTRUcXg=')
print(decoded.hex())
```

Output:

```
766760776558134e157c544b1214107c16531740107c4710401347124d157c12167c177c144b124d151c5e
```

Still not readable. The bytes are in a weird range — not normal ASCII. I thought: maybe XOR encryption?

---

## Step 5 — XOR with the Key from the Memo

The memo said: **"ensure compliance with policy #23"**

The number **23** in hex is `0x17`. But I tried decimal value `35` which is `0x23` (because `#23` means the hex value `0x23`).

```python
key = 0x23
result = bytes([b ^ key for b in decoded])
print(result.decode('ascii'))
```

Output:

```
UDCTF{0m6_wh173_5p4c3_d3c0d1n6_15_4_7h1n6?}
```

**FLAG FOUND!**

---

## Summary

| Step | What I Did | Result |
|------|-----------|--------|
| 1 | Opened file in hex dump, found invisible characters | Trailing `\t` and ` ` on lines |
| 2 | Extracted trailing whitespace | 478 bits of hidden data |
| 3 | Decoded TAB=1, SPACE=0 → ASCII | Base64 string |
| 4 | Base64 decoded | Encrypted bytes |
| 5 | XOR with `0x23` (policy **#23** hint) | **FLAG** |

---

## Tools Used

- `xxd` — hex dump to find invisible bytes
- `Python 3` — custom script for bit extraction and decoding
- `base64` — Python standard library
- Manual XOR — no special tool needed

---

## Key Lesson

> **Always check trailing whitespace in text files.** Normal text editors hide it. Use `xxd`, `cat -A`, or `vi` with `:set list` to see invisible characters. The challenge name and description almost always give you a hint about the technique.

```bash
# Quick check command for any CTF text file:
cat -A suspicious_file.txt | grep -E '\^I| \$'
```

`^I` = tab, and spaces before `$` (end of line) = hidden data.

---

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
