# There’s No Room Left — CTF Writeup

## Challenge Information

- **Challenge name:** There’s no room left
- **Category:** Steganography
- **Attachment:** `flag.txt`
- **Given hash:** `SHA1: 0f425ef8c8922ef1bad11929b317bc9d7b21a73`
- **Flag:** `CIT{ok_maybe_not_plain_sight}`

## Summary

The downloaded text file looked like normal text:

```text
Another year, another steg challenge..
Something-something the flag is hidden in plain sight...
```

At first, the visible sentence did not contain the flag. The hint said the flag was hidden in “plain sight,” so I checked whether the file contained invisible Unicode characters. It did.

The flag was encoded using zero-width / invisible characters inserted between the visible words.

## Initial Inspection

Opening the file normally only showed a normal paragraph:

```bash
cat flag.txt
```

Output:

```text
Another year, another steg challenge.. Something-something the flag is hidden in plain sight, but I'll leave it up to you to see if that really is true or not!
```

Since the file was suspiciously a text file for a steganography challenge, I checked the raw bytes / Unicode characters.

Useful commands:

```bash
xxd flag.txt | head
```

or:

```bash
python3 - <<'PY'
from pathlib import Path
s = Path("flag.txt").read_text(encoding="utf-8")

for i, ch in enumerate(s):
    if ord(ch) > 127:
        print(i, hex(ord(ch)), repr(ch))
PY
```

This revealed many invisible Unicode characters such as:

```text
U+200C
U+200D
U+202C
U+FEFF
```

These characters do not visibly render in most text editors, which is why the file looked normal.

## Encoding Idea

There were four different invisible characters. Four symbols can represent two bits each:

| Character | Unicode Name | Bits |
|---|---|---|
| `U+200C` | Zero Width Non-Joiner | `00` |
| `U+200D` | Zero Width Joiner | `01` |
| `U+202C` | Pop Directional Formatting | `10` |
| `U+FEFF` | Zero Width No-Break Space / BOM | `11` |

So the hidden message can be decoded by:

1. Extracting only the invisible characters.
2. Converting each invisible character into two bits.
3. Grouping the bitstream into bytes.
4. Decoding the bytes as text.

## Solver Script

```python
from pathlib import Path

data = Path("flag.txt").read_text(encoding="utf-8")

mapping = {
    "\u200c": "00",  # Zero Width Non-Joiner
    "\u200d": "01",  # Zero Width Joiner
    "\u202c": "10",  # Pop Directional Formatting
    "\ufeff": "11",  # Zero Width No-Break Space / BOM
}

bits = ""

for ch in data:
    if ch in mapping:
        bits += mapping[ch]

raw = bytes(
    int(bits[i:i+8], 2)
    for i in range(0, len(bits), 8)
    if len(bits[i:i+8]) == 8
)

print(raw)
print(raw.decode("utf-16-be"))
```

## Output

Running the script produced:

```text
b'\x00C\x00I\x00T\x00{\x00o\x00k\x00_\x00m\x00a\x00y\x00b\x00e\x00_\x00n\x00o\x00t\x00_\x00p\x00l\x00a\x00i\x00n\x00_\x00s\x00i\x00g\x00h\x00t\x00}'
CIT{ok_maybe_not_plain_sight}
```

The first raw output has `\x00` before each ASCII character, which indicates the hidden text was effectively encoded as UTF-16 big-endian.

## Flag

```text
CIT{ok_maybe_not_plain_sight}
```

## Takeaway

The challenge hid data using invisible Unicode characters. When a text file looks normal but is used in a steganography challenge, it is worth checking for:

- zero-width characters,
- unusual Unicode codepoints,
- strange byte patterns,
- BOM markers,
- non-printable characters.

In this case, the visible sentence was just a cover, while the real flag was encoded between the words.

---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
