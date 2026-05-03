# Capybara Secret - Writeup

## Challenge
A capybara portrait allegedly hides a secret message.
Flag format: `KubSTU{...}`

## File
- Image: `/media/sf_kali/challenge.jpg`

## Approach
The phrase "look beyond the surface" suggests hidden data rather than visible text, so metadata inspection is a good first step.

### 1. Inspect EXIF metadata
```bash
exiftool /media/sf_kali/challenge.jpg
```

Key finding:
```text
XP Comment : XhoFGH{J0J_1aperq1oyr_pnclon6n}
```

### 2. Decode obfuscated text (ROT13)
The string looks ROT13-encoded.

```bash
printf '%s\n' 'XhoFGH{J0J_1aperq1oyr_pnclon6n}' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Decoded output:
```text
KubSTU{W0W_1ncred1ble_capyba6a}
```

## Final Flag
`KubSTU{W0W_1ncred1ble_capyba6a}`


---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
