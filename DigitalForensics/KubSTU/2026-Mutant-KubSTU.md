# mutant

## Challenge
We are given a PDF (`crypt.pdf`) and told the flag format is `KubSTU{...}`.

## Initial recon
Checked file type and extracted visible text:

```bash
file /media/sf_kali/crypt.pdf
pdftotext /media/sf_kali/crypt.pdf -
```

Visible content was only a short article about cryptography, but it hinted that hidden data exists.

## Finding hidden content
Dumped raw strings from the PDF:

```bash
strings -n 4 /media/sf_kali/crypt.pdf
```

The PDF had two content streams. The second stream (`5 0 obj`) used `/Filter /FlateDecode` and contained an Ascii85-looking blob (`<~ ... ~>`), which is suspicious.

## Decoding stream 5
Decoded Ascii85, then zlib-inflated it:

```python
import re, base64, zlib
b = open('/media/sf_kali/crypt.pdf','rb').read()
s = re.search(rb'5 0 obj.*?stream\n(.*?)\nendstream', b, re.S).group(1).strip()
out = zlib.decompress(base64.a85decode(s, adobe=True))
print(out.decode('latin1'))
```

This revealed many PDF drawing commands of the form:

```text
BT 1 0 0 1 x y Tm (char) Tj ET
```

At `y=100`, characters formed:

```text
KubSTU{pdf_0bj3ct_m4st3r_v2}
```

## Decoys
The hidden stream also included fake/decoy strings:

- `FAKE{this_is_not_the_flag_Otry_harder}`
- `CTF{you_are_close_but_not_yet}`

Only one matches the required format `KubSTU{...}`.

## Final flag
`KubSTU{pdf_0bj3ct_m4st3r_v2}`

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

