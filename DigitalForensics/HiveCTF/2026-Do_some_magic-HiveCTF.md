# Do Some Magic — Writeup

## Challenge

We are given a ZIP archive from malicious emails. The dropped files appear to have been tampered with, and the goal is to recover them.

The challenge title, **Do Some Magic**, hints at checking **file magic bytes** / file headers.

---

## 1. Extract the artifacts

```bash
unzip Do_Some_Magic.zip -d magic
cd magic
ls -la
```

Inside the archive, there are two suspicious files:

```text
flag1.txt
flag2.txt
```

Even though they have `.txt` extensions, the challenge says the files were tampered with, so the extension should not be trusted.

---

## 2. Identify the file type

Run:

```bash
file flag1.txt flag2.txt
```

The files are not detected properly, which suggests their header bytes were corrupted.

Check the beginning of each file:

```bash
xxd -l 64 flag1.txt
xxd -l 64 flag2.txt
```

A normal PNG file should begin with this magic header:

```text
89 50 4E 47 0D 0A 1A 0A
```

In ASCII-ish form, this is:

```text
.PNG....
```

A valid PNG also has an `IHDR` chunk near the start:

```text
49 48 44 52
```

And ends with an `IEND` chunk:

```text
49 45 4E 44
```

The corrupted files looked very close to PNG files, but some magic bytes / chunk names had been damaged.

---

## 3. Repair the PNG headers

Patch the first 8 bytes of each file to the correct PNG signature:

```bash
printf '\x89PNG\r\n\x1a\n' | dd of=flag1.txt bs=1 seek=0 count=8 conv=notrunc
printf '\x89PNG\r\n\x1a\n' | dd of=flag2.txt bs=1 seek=0 count=8 conv=notrunc
```

Then check again:

```bash
file flag1.txt flag2.txt
```

If the PNG chunk names are also damaged, inspect them with:

```bash
xxd flag1.txt | head
xxd flag2.txt | head
```

Patch corrupted chunk names if needed:

```text
IHDR = 49 48 44 52
IEND = 49 45 4E 44
```

After repairing the files, rename them:

```bash
mv flag1.txt flag1.png
mv flag2.txt flag2.png
```

---

## 4. Open the recovered images

```bash
xdg-open flag1.png
xdg-open flag2.png
```

The recovered images contain two parts of the flag.

`flag1.png` shows:

```text
HiveCTF{pl@y1ng_w1th_
```

`flag2.png` shows:

```text
m@g1c_by1e3}
```

Combine both parts:

```text
HiveCTF{pl@y1ng_w1th_m@g1c_by1e3}
```

---

## Flag

```text
HiveCTF{pl@y1ng_w1th_m@g1c_by1e3}
```
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
