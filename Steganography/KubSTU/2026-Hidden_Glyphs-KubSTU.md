# Hidden Glyphs - Steganography/Forensics Writeup

## Challenge
- Category: Steganography / Forensics
- Difficulty: Hard
- File: `/media/sf_kali/stego_challenge.pdf`
- Flag format: `KubSTU{...}`

## Initial Observation
The PDF appears normal when opened. It contains visible text and a hint:

- `Hint: The font hides more than you see...`
- `Each glyph has a width. What do they tell?`

This strongly suggests information is hidden inside font metrics instead of visible text.

## Triage
I checked basic PDF metadata and structure:

```bash
file /media/sf_kali/stego_challenge.pdf
pdfinfo /media/sf_kali/stego_challenge.pdf
exiftool /media/sf_kali/stego_challenge.pdf
```

Key findings:
- PDF 1.7, 1 page, not encrypted
- Keywords include `font` and `encoding`
- No JavaScript or obvious attachments

Then I unpacked the PDF for readable object inspection:

```bash
qpdf --qdf --object-streams=disable /media/sf_kali/stego_challenge.pdf /tmp/stego_pdf/unpacked.pdf
```

## Core Forensic Finding
Inside unpacked objects, page content uses:
- `/F1` (custom `Type3` font)
- `/F2` (Helvetica)

The custom font dictionary (`/Subtype /Type3`) includes a `/Widths` array:

```pdf
12 0 obj
[
  750 1170 980 830 840 850 1230 1160 1210 1120
  510 950 510 950 1020 480 1100 1160 950 1190
  490 1000 1160 1040 530 950 520 1140 510 950
  1160 1140 490 990 1070 1210 1250 500 500 ...
]
endobj
```

The first values immediately map to ASCII if divided by 10:

- `750 -> 75 -> 'K'`
- `1170 -> 117 -> 'u'`
- `980 -> 98 -> 'b'`

So the hidden message is encoded as:
- `character = chr(width / 10)`

The repeated trailing `500` values are padding (`50 -> '2'` if interpreted, but clearly filler here given sequence and context).

## Decode Script
```python
import re

pdf = open('/tmp/stego_pdf/unpacked.pdf', 'r', errors='ignore').read()
arr = re.search(r'12 0 obj\\s*\\[(.*?)\\]\\s*endobj', pdf, re.S).group(1)
widths = [int(x) for x in re.findall(r'\\d+', arr)]

decoded = ''.join(chr(w // 10) for w in widths if w != 500)
print(decoded)
```

Output:

```text
KubSTU{typ3_3_f0nt_w1dth5_4r3_tr1cky}
```

## Final Flag

`KubSTU{typ3_3_f0nt_w1dth5_4r3_tr1cky}`

## Why This Works
- Type3 fonts let authors define custom glyph behavior and metrics.
- Glyph widths are normally rendering metadata, rarely scrutinized.
- The challenge embeds ASCII codes into width values (scaled by 10), hiding data in plain PDF internals.



---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
