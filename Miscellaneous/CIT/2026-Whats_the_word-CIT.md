# What’s the Word? — CTF Writeup

## Challenge Info

- **Challenge name:** What’s the Word?
- **Category:** Forensics
- **Prompt:** `B-b-b-bird, bird, bird! The flag is in the file, but where??`
- **Given SHA1:** `b2b21068c3622d102c62be9b3cc386cc175eff`
- **Final flag:** `CIT{b1rd_1s_th3_w0rd}`

## Summary

The provided file was not a normal text/image file. It was an encrypted Microsoft Office document.  
The clue `B-b-b-bird, bird, bird!` pointed toward the phrase **“bird is the word”**, but the actual password was cracked from the Office encryption hash using John the Ripper.

After cracking the password, the document was decrypted, extracted as a `.docx` archive, and the flag was found inside an embedded image.

## Step 1 — Identify the File

First, inspect the file:

```bash
file file
```

The file behaves like an encrypted Office document rather than a normal readable document. Since encrypted Office files store password verification data, we can extract a John-compatible hash from it.

## Step 2 — Extract the Office Hash

Kali already had John installed, but `office2john.py` was not in `$PATH`, so we located it manually:

```bash
find /usr -name office2john.py 2>/dev/null
```

Output:

```text
/usr/share/john/office2john.py
```

Then we extracted the hash:

```bash
python3 /usr/share/john/office2john.py file > hash.txt
cat hash.txt
```

Output:

```text
file:$office$*2013*100000*256*16*42c71bac48d39fb13c1528f9c39e7b17*4aee5a0a38f0e72f05fd413a96f9b03a*f377ff864c015f83ad20b8bae2cfaceaa24e196932ecde0813adae4306fc131d
```

This confirms the file is protected with Microsoft Office encryption.

## Step 3 — Crack the Password

Use John with `rockyou.txt`:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```

John successfully cracked the password:

```text
q1w2e3r4t5       (file)
```

So the Office document password is:

```text
q1w2e3r4t5
```

## Step 4 — Decrypt the Office File

Install `msoffcrypto-tool` if needed:

```bash
python3 -m pip install msoffcrypto-tool
```

Decrypt the file:

```bash
python3 - << 'PY'
import msoffcrypto

with open("file", "rb") as f:
    office = msoffcrypto.OfficeFile(f)
    office.load_key(password="q1w2e3r4t5")

    with open("decrypted.docx", "wb") as out:
        office.decrypt(out)
PY
```

This produces:

```text
decrypted.docx
```

## Step 5 — Extract the DOCX Contents

A `.docx` file is a ZIP archive, so unzip it:

```bash
unzip decrypted.docx -d docx_out
```

Search the extracted files:

```bash
find docx_out -type f | sort
```

The embedded media files are stored under:

```text
docx_out/word/media/
```

Open the embedded image:

```bash
xdg-open docx_out/word/media/image1.png
```

## Step 6 — Read the Flag

The extracted image contains the flag:

```text
CIT{b1rd_1s_th3_w0rd}
```

## Flag

```text
CIT{b1rd_1s_th3_w0rd}
```

## Takeaway

The main trick was recognizing that the given file was an encrypted Office document.  
Instead of searching the file directly with `strings`, the correct path was:

1. Extract the Office password hash with `office2john.py`.
2. Crack the hash with John the Ripper.
3. Decrypt the document using the recovered password.
4. Extract the `.docx` contents.
5. Inspect embedded media for the hidden flag.

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

