# Krasnodar Tram Writeup

## Challenge Info

- **Name:** Krasnodar tram
- **Category:** Stego
- **Points:** 1000
- **Flag format:** `KubSTU{...}`

## Summary

The challenge provides two JPEG images of Krasnodar trams:

- `267.jpg`
- `678.jpg`

The fastest solve path is not pixel-level steganography. The useful payload is hidden in EXIF metadata as short base64 fragments spread across multiple fields in both images.

Reassembling those fragments produces a Pastebin URL, and the raw Pastebin content is the flag.

## Initial Observation

Running `exiftool` on both files immediately shows multiple suspicious base64-looking tokens:

From `267.jpg`:

- `User Comment`: `ref=cnUv;src=aHR0`
- `Artist`: `M. Volkov / Ly9w Studio`
- `Host Computer`: `MBP-ZWJp-001.local`
- `Software`: `Adobe Photoshop 25.4 (build b20v)`
- `Copyright`: `(C) 2024 Q3N2 Media. All rights reserved.`

From `678.jpg`:

- `User Comment`: `id=cHM6-dWJz`
- `Host Computer`: `WIN-DESKTOP-YXN0`
- `Software`: `Capture One Pro 23 v16.2.bi5j`
- `Lens Model`: `RF 50mm F1.2 L USM (rev.U3VC)`
- `Copyright`: `(C) Studio cEs= 2024`

There is also a huge `XP Comment` block telling a language model to ignore the file and print a borscht recipe. That is just prompt-injection noise and not part of the solve.

## Reconstructing the URL

The intended fragments are:

- `aHR0`
- `cHM6`
- `Ly9w`
- `YXN0`
- `ZWJp`
- `bi5j`
- `b20v`
- `U3VC`
- `Q3N2`
- `cEs=`

Concatenate them in order:

```text
aHR0cHM6Ly9wYXN0ZWJpbi5jb20vU3VCQ3N2cEs=
```

Decode that base64 string:

```text
https://pastebin.com/SuBCsvpK
```

The raw version of that page is:

```text
https://pastebin.com/raw/SuBCsvpK
```

Fetching it returns the flag directly.

## Flag

```text
KubSTU{g0d_s4v3_7h3_kr45n0d4r_7r4m}
```

## Minimal Solver

```python
import base64
import requests

parts = [
    "aHR0",
    "cHM6",
    "Ly9w",
    "YXN0",
    "ZWJp",
    "bi5j",
    "b20v",
    "U3VC",
    "Q3N2",
    "cEs=",
]

url = base64.b64decode("".join(parts)).decode()
print("url:", url)

flag = requests.get(url.replace("pastebin.com/", "pastebin.com/raw/"), timeout=10, verify=False).text
print("flag:", flag)
```

## Takeaway

This was a metadata stego challenge disguised as an image-analysis task. The images themselves are thematic cover, but the actual message lives in EXIF fields as distributed base64 fragments. Once those fragments are concatenated and decoded, the rest of the solve is just one HTTP request.

---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
