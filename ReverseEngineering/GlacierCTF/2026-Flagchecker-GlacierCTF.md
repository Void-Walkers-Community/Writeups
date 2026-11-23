# Flagchecker: Writeup

**Category:** Beginner reversing | **Author:** LosFuzzys | **Flag:** `gctf{cf776a78f2488869cebfefd87a3e6a9fd582b9e556d0779216c5bb5b45ea4a70}`

## 1. Triage

The archive contains `challenge` and a `sha256sum` file. The binary is an x86-64 ELF, dynamically linked, **not stripped**, and built with debug info. That means `main` and the data symbol `flags` show up directly in `nm`:

```
0000000000401186 T main
0000000000402020 R flags      <- in .rodata
```

`strings` shows only the prompt `Enter flag:` and the two result messages, so the flag is not stored as plain text. The `.rodata` section runs from `0x402020` to `0x550420`, about 1.37 MB. That is exactly 1337 × 256 × 4 bytes, so `flags` is a 1337 × 256 table of 32-bit integers.

## 2. Reading `main`

Reconstructed from the disassembly:

```c
extern uint32_t flags[1337][256];

int main() {
    char buf[256] = {0};
    int32_t in[256] = {0};
    uint32_t acc = 0;

    puts("Enter flag: ");
    fgets(buf, 255, stdin);

    for (size_t i = 0; i < 256; i++) {
        if (buf[i] == '\n') { in[i] = 0; break; }
        in[i] = (int8_t)(buf[i] ^ 0x12);
    }

    for (size_t r = 0; r <= 0x538; r++) {      // 1337 rows
        acc = 0;
        for (size_t j = 0; j < 256; j++) {
            if (in[j] == 0) break;
            acc ^= flags[r][j] ^ in[j];
        }
        if (acc == 0) break;                   // some row matched
    }
    puts(acc == 0 ? "The flag is valid!" : "this flag is unfortunately invalid :(");
}
```

The check is as follows:
1. Each input byte is XORed with `0x12`.
2. For every row of the table, it XORs together all `flags[r][j] ^ in[j]` over the input length.
3. The input is valid if any row gives 0.

Taking one row at a time, the check is the XOR of that row's first *n* entries compared against the XOR of the input's *n* entries. It is a checksum rather than a character-by-character comparison, and the table is bigger than any single flag needs. That suggests the table rows are themselves the candidate flags.

## 3. Decoding the table

Dumping the first entries of row 0 gives `0x74 0x7e 0x75 0x69 …`. XORing each with `0x12` gives `f l g {`, the same key the program applies to the input. So each row is a flag with every byte XORed by `0x12`, terminated by a zero.

```python
import struct
d = open('challenge','rb').read()
t = struct.unpack('<%dI' % (1337*256), d[0x2020:0x2020 + 1337*256*4])

rows = []
for r in range(1337):
    s = ''
    for x in t[r*256:(r+1)*256]:
        if x == 0: break
        s += chr(x ^ 0x12)
    rows.append(s)
```

Some example rows:

```
0 flg{b2f47953db4df41f5da9eb0a3ea335a224fd023502a5e8b874266329b0291f63}
2 los{ac45d687bad84c638bf6360d91070d8c41aa0c3ac1c7761a501466380dcddfcf}
4 sec{5741cb0f02694d505a79ba15ca6796a9536b6da2f5f209597e8714b113ab5c22}
8 ctf{1e9d1c2a5bf429b1cd6b4f8cdaf45c919bd34ffaf7e4fbecf0f487c6ebd6b874}
```

Every row is a decoy flag with a made-up prefix (`flg{`, `los{`, `sec{`, `ctf{`) and 64 hex characters. The binary would accept any of these, since each one trivially matches its own row. Only one row is the real flag.

## 4. Finding the real flag

The real flag has the CTF's prefix, `gctf{`. Filtering the decoded rows by length and prefix:

```python
for i, s in enumerate(rows):
    if len(s) != 69: print(i, s)
# 706 gctf{cf776a78f2488869cebfefd87a3e6a9fd582b9e556d0779216c5bb5b45ea4a70}
```

The 1336 decoys are 69 characters long. Row 706 is 70 characters because `gctf{` is one character longer than the 4-character decoy prefixes. It is also the only row starting with `gctf`.

## 5. Verification

```
$ echo 'gctf{cf776a78f2488869cebfefd87a3e6a9fd582b9e556d0779216c5bb5b45ea4a70}' | ./challenge
Enter flag:
... (10 lines left)
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
