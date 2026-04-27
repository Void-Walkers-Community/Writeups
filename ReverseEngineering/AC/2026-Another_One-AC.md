# Another One Writeup

## Challenge

- Name: `Another One`
- Category: `Rev`
- Points: `100`
- Author: `grdnero`

## Summary

The challenge provides a single file named `executable`, but standard tooling does not recognize it as a normal binary:

```text
$ file executable
executable: data
```

At first glance that suggests packing, encryption, or a custom container. The real trick is simpler: the file is a valid ELF whose contents have been permuted in fixed-size blocks.

The solve path is:

1. detect that the ELF header is present but scrambled
2. identify the block permutation
3. rebuild the executable
4. reverse the input-checking logic
5. recover the accepted key
6. hash the key into the required flag format

## Step 1: Inspect the scrambled file

A quick hex dump shows something very suspicious near the beginning:

```text
00000000: 0000 0000 0000 0000 0001 0102 464c 457f
```

The bytes `46 4c 45 7f` are `FLE\x7f`, which looks like the ELF magic `\x7fELF` written backwards inside a larger pattern.

`strings` gives the same signal. Many symbols appear reversed:

```text
D-xunil-dl/46bil/
rats_cbil__
niam_tgered_MTI_
```

So the file is not random data. It contains a normal ELF with bytes reversed in a structured way.

## Step 2: Identify the transformation

Testing simple hypotheses quickly shows that reversing each 16-byte block restores a valid executable.

Minimal restoration script:

```python
from pathlib import Path

b = Path("executable").read_bytes()
out = bytearray()

for i in range(0, len(b), 16):
    out.extend(b[i:i+16][::-1])

Path("restored").write_bytes(out)
```

After that:

```text
$ file restored
restored: ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped
```

Running the repaired binary prints a Khaled-themed prompt and asks for a `record key`.

## Step 3: Locate the validation logic

The restored binary is stripped, but the validation path is easy to isolate because the success and failure strings are embedded in `.rodata`:

- `record key>`
- `[+] DJ Khaled says: we the best music`
- `[-] DJ Khaled says: they did not believe in us.`

Disassembly around that path shows three important behaviors:

1. the program trims trailing `\\r` and `\\n`
2. it requires the cleaned input length to be exactly `0x1b` bytes, or `27`
3. it copies the input into a temporary buffer in reverse order before validating it

That reverse copy is performed by a recursive helper. So if the reversed buffer is:

```text
no1s1vru0ys10sytpm3s13gd1rf
```

then the actual user input must be:

```text
fr1dg31s3mptys01sy0urv1s1on
```

## Step 4: Understand the checker

The core checker is another recursive function. It walks over the 27 reversed bytes and updates:

- a 32-bit rolling accumulator
- a one-byte error flag that must stay zero

For each index `i`, it:

1. mixes the current byte into a rolling 32-bit state
2. rotates the current input byte by an index-dependent amount
3. xors it with:
   - a byte from the string `DJ KHALED!`
   - a 27-byte opaque table in `.rodata`
   - one byte extracted from the updated accumulator
4. ORs the result into a flag byte

The recursive function returns success only if:

- the final flag byte is `0`
- the final accumulator equals `0xf459236d`

That means every byte must satisfy a local constraint, but the accumulator creates state across all positions. A naive greedy solve fails at a few ambiguous positions, so the clean solution is a small backtracking search over the 27 bytes.

## Step 5: Reconstruct the key

The following solver reproduces the binary logic closely enough to recover the accepted reversed buffer and then flips it back into the real input key.

```python
from pathlib import Path
import hashlib

b = Path("restored").read_bytes()

table_a = b[0x5A4F:0x5A4F + 27]
table_b = b[0x5A34:0x5A34 + 27]

C = 0x045D9F3B
TARGET = 0xF459236D
START = 0x50414C31


def rol32(x, n):
    n &= 31
    if n == 0:
        return x & 0xFFFFFFFF
    return ((x << n) | (x >> (32 - n))) & 0xFFFFFFFF


def rol8(x, n):
    n %= 8
    if n == 0:
        return x & 0xFF
    return ((x << n) | (x >> (8 - n))) & 0xFF


rots = []
idxs = []
for i in range(27):
    a = (37 * i) >> 8
    r = (i - a) & 0xFF
    r >>= 1
    r = (r + a) & 0xFF
    r >>= 2
    r = (r + i + 1) & 0xFF
    rots.append(r)

    idx = (i - 3 * (((i * 0xAB) >> 9) & 0x7C)) & 0xFF
    idxs.append(idx)


memo = {}


def dfs(i, acc, flag):
    key = (i, acc, flag)
    if key in memo:
        return memo[key]

    if i == 27:
        memo[key] = b"" if acc == TARGET and flag == 0 else None
        return memo[key]

    for ch in range(256):
        tmp = (i * 0x31 + ch) & 0xFFFFFFFF
        new_acc = ((tmp * C) & 0xFFFFFFFF) ^ rol32(acc, 5)

        state_byte = (new_acc >> ((i * 8) & 31)) & 0xFF
        v = rol8(ch, rots[i]) ^ table_a[idxs[i]] ^ table_b[i] ^ state_byte
        new_flag = (flag | v) & 0xFF

        if new_flag != 0:
            continue

        rest = dfs(i + 1, new_acc, new_flag)
        if rest is not None:
            memo[key] = bytes([ch]) + rest
            return memo[key]

    memo[key] = None
    return None


reversed_buf = dfs(0, START, 0)
key = reversed_buf[::-1].decode()

print("reversed buffer:", reversed_buf.decode())
print("key:", key)
print("flag:", f"CTFAC{{{hashlib.sha256(key.encode()).hexdigest()}}}")
```

Running that solver yields:

```text
reversed buffer: no1s1vru0ys10sytpm3s13gd1rf
key: fr1dg31s3mptys01sy0urv1s1on
flag: CTFAC{19de747e6bafb8e23a4e050e61213679d1c3865f7e854c217a5fdcbdd7fcb647}
```

## Step 6: Verify against the binary

Feeding the recovered key into the repaired program produces the success message:

```text
record key> [+] DJ Khaled says: we the best music
```

So the accepted key is:

```text
fr1dg31s3mptys01sy0urv1s1on
```

## Flag

```text
CTFAC{19de747e6bafb8e23a4e050e61213679d1c3865f7e854c217a5fdcbdd7fcb647}
```

## Minimal Reproduction

These commands are enough to reproduce the core solve flow:

```bash
file executable
xxd -l 64 executable
python3 - <<'PY'
from pathlib import Path
b = Path("executable").read_bytes()
out = bytearray()
for i in range(0, len(b), 16):
    out.extend(b[i:i+16][::-1])
Path("restored").write_bytes(out)
PY

file restored
strings -a restored | rg 'record key|Khaled|we the best|believe'
objdump -d -Mintel restored | sed -n '/f240/,/f430/p'
python3 solve.py
printf 'fr1dg31s3mptys01sy0urv1s1on\n' | ./restored
```

## Takeaway

This is a compact reverse challenge with two clean ideas:

- the executable is hidden by a fixed 16-byte block reversal instead of real encryption
- the key check combines per-byte constraints with a rolling state, so local reasoning alone is not enough

Once the ELF is repaired, the rest of the challenge reduces to ordinary static analysis and a small stateful search. The Khaled-themed clues about “another byte, another row, another direction” fit the intended solve nicely: the bytes were there all along, just facing the wrong way.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
