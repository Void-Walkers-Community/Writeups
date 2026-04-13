<img width="659" height="313" alt="image" src="https://github.com/user-attachments/assets/10bfba43-ef1e-4aa8-a800-911fd2c0c701" /># Penguin_Steg Writeup

## Challenge Summary

We are given a JPEG image named `Penguin_Steg.jpg` and told:

- there is something hidden in the image
- "the key is the last word"
- flag format is `DawgCTF{}`

The image itself is a penguin, which strongly suggests Linux/Tux.

## Initial Analysis

Basic checks showed the file was a normal JPEG with no obvious appended archive or metadata clue:

```bash
file /media/sf_kali/Penguin_Steg.jpg
exiftool /media/sf_kali/Penguin_Steg.jpg
binwalk /media/sf_kali/Penguin_Steg.jpg
```

The important clue came from `steghide`:

```bash
steghide info /media/sf_kali/Penguin_Steg.jpg
```

This indicated the JPEG had capacity for embedded data, which is a strong sign that `steghide` was used.

## Hint Interpretation

The extra hint was:

> Search up Linux (To make a duplicate + Spaghetti is a type of what?)

At first, `copy + pasta = copypasta` seems tempting, but that was not the correct passphrase.

The Linux-specific part of the hint points to the classic "GNU/Linux" copypasta, where people insist that "Linux" should really be called `GNU/Linux`.

That makes `GNU/Linux` the key.

## Extraction

Use `steghide` with the passphrase `GNU/Linux`:

```bash
steghide extract -sf /media/sf_kali/Penguin_Steg.jpg -p 'GNU/Linux' -xf out
```

This successfully extracts the hidden file.

## Recovering the Flag

The extracted file is plain ASCII text:

```bash
file out
strings out
```

Contents:

```text
DawgCTF{UmActu@LlYIT$GnUL!nUX}
```

## Final Flag

```text
DawgCTF{UmActu@LlYIT$GnUL!nUX}
```
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


