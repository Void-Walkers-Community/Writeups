# HoneyPot Writeup

## Challenge

> We pulled this binary off a threat actor's server. It's asking for a passphrase, figure out what it wants.

We are given a single Linux ELF binary named `honeypot`. The goal is to reverse the password/flag check.

---

## 1. Basic file inspection

```bash
$ file honeypot
honeypot: ELF 64-bit LSB pie executable, x86-64, dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, stripped
```

The binary is:

- 64-bit ELF
- PIE enabled
- dynamically linked
- stripped, so there are no useful function names

Running `strings` gives some useful hints:

```bash
$ strings -a honeypot
HiveCTF{
=== HoneyPot ===
Enter the flag:
Input error.
Correct! You got it.
Wrong flag. Keep reversing!
```

So the flag likely starts with:

```text
HiveCTF{
```

---

## 2. Finding the checker

Disassembling with `objdump` shows the main input flow:

```bash
objdump -d -M intel honeypot
```

The program:

1. Prints the banner.
2. Reads input using `fgets`.
3. Removes the trailing newline.
4. Calls a checker function.
5. Prints either success or failure.

The important logic is inside the checker around `0x11e9`.

The first checks are simple:

```asm
cmp DWORD PTR [rsp+0xc],0x16
jne fail

strncmp(input, "HiveCTF{", 8)
cmp BYTE PTR [input+0x15], 0x7d
```

This means the full flag length must be `0x16`, or 22 bytes.

Since `HiveCTF{` is 8 bytes and `}` is 1 byte:

```text
HiveCTF{?????????????}
        13 chars
```

So the unknown inner part is 13 bytes long.

---

## 3. Extracting the inner flag

The binary copies the 13 inner characters from the input onto the stack:

```asm
mov rax, QWORD PTR [input+0x8]
mov QWORD PTR [rsp+0x33], rax

mov rax, QWORD PTR [input+0xd]
mov QWORD PTR [rsp+0x38], rax
```

Because the second copy overlaps the first, the final copied region is:

```text
input[8:21]
```

That is exactly the 13 characters inside the braces.

---

## 4. The keystream

The checker uses the first 4 bytes of the inner flag as a big-endian seed:

```asm
shl eax, 0x8
or  eax, current_byte
```

Then it applies a xorshift32-style PRNG:

```asm
x ^= x << 13
x ^= x >> 17
x ^= x << 5
```

For each of the 13 bytes, it stores the low byte of the PRNG output as a keystream byte.

In Python, that is:

```python
def xs32(x):
    x &= 0xffffffff
    x ^= (x << 13) & 0xffffffff
    x ^= x >> 17
    x ^= (x << 5) & 0xffffffff
    return x & 0xffffffff

seed = int.from_bytes(inner[:4], "big")
stream = []
for _ in range(13):
    seed = xs32(seed)
    stream.append(seed & 0xff)
```

---

## 5. AES S-box transform

In `.rodata`, the binary contains the AES S-box starting at `0x20c0`:

```text
63 7c 77 7b f2 6b 6f c5 30 01 67 2b fe d7 ab 76 ...
```

The checker transforms each inner character like this:

```python
transformed[i] = AES_SBOX[inner[i]] ^ keystream[i]
```

Then it compares the 13 transformed bytes against a target array at `0x20a0`:

```text
03 f1 a0 b4 28 11 86 93 66 32 5e 02 cf
```

So the check is:

```python
AES_SBOX[inner[i]] ^ keystream[i] == target[i]
```

Because the keystream depends on the first 4 bytes of the inner flag, we can brute force those first 4 bytes, generate the keystream, invert the S-box, and recover the full inner flag.

---

## 6. Solver

```python
from itertools import product
import string

TARGET = bytes.fromhex("03 f1 a0 b4 28 11 86 93 66 32 5e 02 cf")

SBOX = bytes.fromhex(
    "637c777bf26b6fc53001672bfed7ab76"
    "ca82c97dfa5947f0add4a2af9ca472c0"
    "b7fd9326363ff7cc34a5e5f171d83115"
    "04c723c31896059a071280e2eb27b275"
    "09832c1a1b6e5aa0523bd6b329e32f84"
    "53d100ed20fcb15b6acbbe394a4c58cf"
    "d0efaafb434d338545f9027f503c9fa8"
    "51a3408f929d38f5bcb6da2110fff3d2"
    "cd0c13ec5f974417c4a77e3d645d1973"
    "60814fdc222a908846eeb814de5e0bdb"
    "e0323a0a4906245cc2d3ac629195e479"
    "e7c8376d8dd54ea96c56f4ea657aae08"
    "ba78252e1ca6b4c6e8dd741f4bbd8b8a"
    "703eb5664803f60e613557b986c11d9e"
    "e1f8981169d98e949b1e87e9ce5528df"
    "8ca1890dbfe6426841992d0fb054bb16"
)

INV_SBOX = [0] * 256
for i, b in enumerate(SBOX):
    INV_SBOX[b] = i


def xs32(x):
    x &= 0xffffffff
    x ^= (x << 13) & 0xffffffff
    x ^= x >> 17
    x ^= (x << 5) & 0xffffffff
    return x & 0xffffffff


def keystream(seed):
    out = []
    x = seed
    for _ in range(13):
        x = xs32(x)
        out.append(x & 0xff)
    return bytes(out)


# The flag body uses normal CTF flag characters.
alphabet = string.ascii_lowercase + string.digits + "_!"

for prefix_tuple in product(alphabet.encode(), repeat=4):
    prefix = bytes(prefix_tuple)
    seed = int.from_bytes(prefix, "big")
    ks = keystream(seed)

    inner = bytes(INV_SBOX[t ^ k] for t, k in zip(TARGET, ks))

    if inner.startswith(prefix) and all(chr(c) in alphabet for c in inner):
        print(f"HiveCTF{{{inner.decode()}}}")
        break
```

Running it gives:

```bash
$ python3 solve.py
HiveCTF{r3v_7h3_4lg0!}
```

---

## 7. Verification

```bash
$ ./honeypot
=== HoneyPot ===
Enter the flag: HiveCTF{r3v_7h3_4lg0!}
Correct! You got it.
```

---

## Flag

```text
HiveCTF{r3v_7h3_4lg0!}
```

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
