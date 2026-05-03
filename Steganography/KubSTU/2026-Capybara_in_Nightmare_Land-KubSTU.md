# Capybara in Nightmare Land (Stego) Writeup

## Challenge
A PNG image (`capybara_nightmare.png`) hides a secret message.
Flag format: `KubSTU{...}`

## 1) Initial file triage
Checked type/metadata:

```bash
file capybara_nightmare.png
exiftool capybara_nightmare.png
pngcheck -vt capybara_nightmare.png
```

Key finding:
- `exiftool` warning: **Trailer data after PNG IEND chunk**
- `pngcheck` also reported **additional data after IEND**

This strongly suggests appended hidden content.

## 2) Detect and extract appended payload
Used `binwalk`:

```bash
binwalk capybara_nightmare.png
```

Important output:
- ZIP archive found near the end of PNG
- Contains:
  - `README.txt`
  - `encrypted_flag.bin`

Extracted with:

```bash
binwalk -e capybara_nightmare.png
```

## 3) Read hint and encrypted bytes
`README.txt` states:
- flag is XOR encrypted
- key is hidden in image pixels
- hint: **LSB**
- key length: **19**

Encrypted bytes:

```text
0544053b20384f3a03333a6b3d49334b6f71573e482f09370605004e
```

## 4) Recover LSB key
Ran zsteg against PNG:

```bash
zsteg -a capybara_nightmare.png
```

Relevant hit:

```text
b1,rgb,lsb,xy .. text: "N1ghtm4r3_C4py_2026"
```

Recovered key:

```text
N1ghtm4r3_C4py_2026
```

(length = 19, matches hint)

## 5) XOR decrypt
Used Python:

```python
enc = bytes.fromhex("0544053b20384f3a03333a6b3d49334b6f71573e482f09370605004e")
key = "N1ghtm4r3_C4py_2026"
flag = ''.join(chr(b ^ ord(key[i % len(key)])) for i, b in enumerate(enc))
print(flag)
```

Output:

```text
KubSTU{H0ly_M0ly_CapyHaCk1r}
```

## Final Flag
`KubSTU{H0ly_M0ly_CapyHaCk1r}`


---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
