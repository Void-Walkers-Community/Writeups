# Firmware Whisper Writeup

## Challenge

- Name: `firmware whisper`
- Category: `REV`
- Points: `100`
- Author: `gR3nn`

## Summary

The challenge ships two files:

- `firmware.wfw`
- `recovered_notes.txt`

The notes immediately give away two important facts:

- the firmware is wrapped in a custom `LZWQ` container
- the recovered ROM uses an undocumented `WHISK-1` architecture

The `WHISK-1` notes are useful context, but they are also the main distraction. The intended shortcut is not full ISA recovery. After decompressing the container, the image contains:

- a visible 30-byte unlock token at offset `0x22`
- a success string `UNLOCK sequence accepted`
- a separate 71-byte opaque blob at `0x100..0x146`

The `deadbeefcafebabe...` token is only an in-firmware authentication string. The real flag is the 71-byte blob. Its length is exactly:

```text
len("CTFAC{") + 64 + len("}") = 71
```

That makes it a perfect candidate for an encrypted `CTFAC{64 hex chars}` flag. Brute forcing a tiny XOR keystream generator against the known `CTFAC{` prefix recovers the unique valid plaintext.

## Step 1: Decompress the `WFW` container

A quick dump of the file header shows a custom format:

```text
00000000: 57 46 57 01 01 00 a7 4f 00 00 00 e8 00 00 00 00
00000010: 00 00 ba 0b ...
```

The important observations are:

- magic is `WFW`
- the decompressed size is `0x0000e800`
- the actual compressed payload starts at offset `0x14`, not `0x40`

The recovered notes describe the token classes:

- literal: tag `0x00`, followed by one byte
- short back-reference: tag `0x40`, followed by one packed byte
- long back-reference: tag `0x80`, followed by a big-endian packed word
- RLE: tag `0xc0`, followed by `value, count`
- terminator: `0xff`

Two implementation details matter:

- back-reference copies must be performed byte-by-byte so overlap works correctly
- `dist = 0` is valid and means “repeat the previous byte”

Using that decoder on `firmware.wfw[0x14:]` produces an exact `0xe800`-byte image.

## Step 2: Inspect the decompressed image

Running `strings` on the decompressed ROM gives a clean split between firmware-like text and opaque data:

```text
147 Booting RTOS v2.1.4
15b [sys] Mounting SPIFFS partition...
17e [sys] SPIFFS mount failed, returning to fallback ROM
1d6 init_sensor
1e2 auth_check
1ed debug_mode
242 UNLOCK sequence accepted
25b -----BEGIN PUBLIC KEY-----
2a7 AES-256-CBC S-Box Configuration:
2c8 [update] OTA firmware update triggered...
```

The obvious first lead is the token at offset `0x22`:

```text
deadbeefcafebabedeadbeefcafebabedeadbeefcafebabedeadbeefcafe
```

That value sits right next to `UNLOCK sequence accepted`, so it is very tempting to submit it directly or wrap it as a flag. That path is wrong. It is just an in-universe unlock sequence embedded in the ROM image.

## Step 3: Focus on the 71-byte blob

The more interesting region is the standalone 71-byte block at `0x100..0x146`:

```text
a8662bcdfc0d5649ebccacf2128f6f998af6d8a9b93fd7b2
92e8869cc17a6b3dbf673ace69f735a3457f03d2a5eb9f7b
6dd468841612679077c8e0a1a608cc101b479cffc984dc
```

That block has three properties that make it much more likely to be the flag payload:

1. it is isolated from the surrounding string table
2. it is high-entropy compared to nearby firmware text
3. its length is exactly the size of a `CTFAC{64-hex}` string

So the problem reduces to: find a simple reversible transform that turns those 71 bytes into printable ASCII beginning with `CTFAC{`.

## Step 4: Recover the keystream

Assume the plaintext starts with:

```text
CTFAC{
```

Since the blob decrypts cleanly under XOR, the first six keystream bytes are immediately known:

```text
keystream[i] = ciphertext[i] ^ plaintext[i]
```

Searching all byte-sized affine recurrences of the form:

```text
k[n + 1] = (a * k[n] + c) mod 256
```

for parameters `a, c in [0, 255]` gives a unique match:

```text
a = 109
c = 35
```

So the full keystream is:

```text
k[0] = blob[0] ^ ord('C')
k[n + 1] = (109 * k[n] + 35) & 0xff
plaintext[n] = blob[n] ^ k[n]
```

Decrypting all 71 bytes yields:

```text
CTFAC{79869f51fa14e569f21bc86425457b6a436e6fb56c665894604bee1f58051362}
```

That string matches the expected flag format exactly and was accepted by the challenge platform.

## Flag

```text
CTFAC{79869f51fa14e569f21bc86425457b6a436e6fb56c665894604bee1f58051362}
```

## Minimal Reproduction

This script reproduces the solve directly from the original `firmware.wfw` attachment.

```python
from pathlib import Path


def decompress_wfw(path: str) -> bytes:
    data = Path(path).read_bytes()
    assert data[:3] == b"WFW"

    out_len = int.from_bytes(data[10:14], "little")
    comp = data[0x14:]

    out = bytearray()
    i = 0

    while i < len(comp) and len(out) < out_len:
        tag = comp[i]
        i += 1

        if tag == 0xFF:
            break

        if tag == 0x00:
            out.append(comp[i])
            i += 1
            continue

        if tag == 0x40:
            packed = comp[i]
            i += 1
            dist = packed >> 4
            length = (packed & 0x0F) + 3

            for _ in range(length):
                src = len(out) - dist if dist else len(out) - 1
                out.append(out[src])
            continue

        if tag == 0x80:
            packed = int.from_bytes(comp[i:i + 2], "big")
            i += 2
            dist = packed >> 4
            length = (packed & 0x0F) + 3

            for _ in range(length):
                src = len(out) - dist if dist else len(out) - 1
                out.append(out[src])
            continue

        if tag == 0xC0:
            value = comp[i]
            count = comp[i + 1]
            i += 2
            out.extend([value] * count)
            continue

        raise ValueError(f"unknown tag: 0x{tag:02x}")

    if len(out) != out_len:
        raise ValueError(f"decoded length {len(out):#x} != expected {out_len:#x}")

    return bytes(out)


rom = decompress_wfw("firmware.wfw")

# The obvious unlock sequence at 0x22 is a decoy:
# deadbeefcafebabedeadbeefcafebabedeadbeefcafebabedeadbeefcafe

blob = rom[0x100:0x147]
prefix = b"CTFAC{"

prefix_keystream = bytes(c ^ p for c, p in zip(blob, prefix))

solution = None
for a in range(256):
    for c in range(256):
        ks = [prefix_keystream[0]]
        for _ in range(len(blob) - 1):
            ks.append((a * ks[-1] + c) & 0xFF)

        if bytes(ks[:len(prefix_keystream)]) != prefix_keystream:
            continue

        plain = bytes(b ^ k for b, k in zip(blob, ks))

        if (
            plain.startswith(b"CTFAC{")
            and plain.endswith(b"}")
            and len(plain) == 71
            and all(chr(x) in "0123456789abcdef" for x in plain[6:-1])
        ):
            solution = plain.decode()
            print(solution)
            raise SystemExit

if solution is None:
    raise RuntimeError("flag not found")
```

## Takeaway

The custom ISA notes are real, but they are not the shortest route to the flag. The decisive clues are:

- the payload starts at `0x14`
- the `deadbeef...` sequence is only an application token
- the 71-byte blob has the exact size of the final flag
- a tiny XOR keystream search is enough to recover the plaintext

This is a good example of a reversing challenge where structural triage beats full emulation.
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
