# Are ya winning, son? — Steganography Writeup

## Challenge

We were given a JPEG image named `challenge.jpg`.

Challenge text:

> Well... is he? It almost feels like we're breaking the fourth wall ;)

Provided SHA1:

```text
1a9accb2f56d4cf2594128aa55875dc7bde5774b
```

The visible image only shows the meme text:

```text
ARE YA WINNING, SON?
"CLAUDE, SOLVE CTF@CIT AND DON'T MAKE ANY MISTAKES"
```

At first glance, this looks like it might be prompt-injection themed, but the category is **Steganography**, so the real payload should be hidden inside the image file.

---

## Initial Checks

First, verify the file hash:

```bash
sha1sum challenge.jpg
```

Output:

```text
1a9accb2f56d4cf2594128aa55875dc7bde5774b  challenge.jpg
```

Then check the file type:

```bash
file challenge.jpg
```

It is a normal-looking JPEG image.

Basic checks such as metadata, appended files, and strings do not immediately reveal the flag:

```bash
exiftool challenge.jpg
strings challenge.jpg | head
binwalk challenge.jpg
```

There is no obvious ZIP, PNG, or flag appended after the JPEG end marker.

---

## Key Observation

The important clue appears when decoding the JPEG more strictly.

Some JPEG tools complain about extra data inside the JPEG scan, with a warning similar to:

```text
Corrupt JPEG data: 8458 extraneous bytes before marker 0xd9
```

This means there are **8458 suspicious bytes before the final JPEG end marker**:

```text
FF D9
```

In a normal JPEG, `FFD9` marks the End of Image. However, in this challenge, there is another hidden JPEG scan stream placed before the final `FFD9`.

So instead of looking after the JPEG, we need to carve hidden data from inside the JPEG entropy-coded scan data.

---

## JPEG Structure Idea

A JPEG generally looks like this:

```text
FFD8 ... headers ... FFDA ... scan data ... FFD9
```

Where:

| Marker | Meaning |
|---|---|
| `FFD8` | Start of Image |
| `FFDA` | Start of Scan |
| `FFD9` | End of Image |

The visible image uses one scan stream.

The hidden flag image is stored as another scan stream, but it does not include its own full JPEG header. Therefore, we can reuse the original JPEG header and combine it with the hidden scan data.

---

## Exploit / Solve Script

For this file, the Start of Scan header ends at offset `623`.

The hidden scan data is the `8458` extraneous bytes before the final `FFD9`.

We rebuild a new JPEG using:

```text
original JPEG header + hidden scan data + FFD9
```

Solve script:

```python
from pathlib import Path

data = Path("challenge.jpg").read_bytes()

# The JPEG header up to and including the Start of Scan header.
# This offset is specific to this challenge file.
header = data[:623]

# Hidden scan stream: 8458 extraneous bytes before the final FFD9.
hidden = data[-8460:-2]

# Rebuild the hidden image.
Path("flag.jpg").write_bytes(header + hidden + b"\xff\xd9")

print("wrote flag.jpg")
```

Run it:

```bash
python3 solve.py
```

Then open the recovered image:

```bash
xdg-open flag.jpg
```

---

## Result

The recovered JPEG shows the flag:

```text
CIT{pls_d0nt_b3_l1k3_th1s_guy}
```

---

## Flag

```text
CIT{pls_d0nt_b3_l1k3_th1s_guy}
```

---

## Summary

The challenge hid a second JPEG scan stream inside the original JPEG file. The payload was not stored after the image end marker, so simple appended-data carving did not work.

The solve was to notice the JPEG decoder warning about:

```text
8458 extraneous bytes before marker 0xd9
```

Then carve those bytes, reuse the original JPEG header, append the JPEG EOI marker, and recover the hidden flag image.

---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
