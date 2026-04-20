# The Evil Files — CTF Writeup

## Challenge Information

- **Challenge:** The Evil Files
- **Category:** Forensics
- **Description:** `Dr. Evil be dreamin and schemin`
- **Given file:** `challenge.pdf`

## Objective

We were given a PDF file and needed to recover the hidden flag.

## Initial Analysis

The challenge provided a single file:

```bash
challenge.pdf
```

Since the file was a PDF, the first thing to do was inspect the visible text and metadata. PDF challenges often hide flags in places such as:

- visible document text
- copied/extracted text
- metadata fields
- annotations
- embedded files
- email-style headers
- hidden layers or objects

## Extracting Text from the PDF

A quick way to inspect the readable contents is to use `pdftotext`:

```bash
pdftotext challenge.pdf -
```

This outputs the text content of the PDF directly to the terminal.

The extracted content looked like an email:

```text
FROM: laser.shark.master@villainhq.net
TO: tiny.turmoil@domination.co
CC: CIT{m0j0_eng4g3d}
Subject: RE: Plan to take over the world
Date: Thur, 15 April 2026 07:30:12 +0000
```

The flag was hidden in the `CC` field of the email header.

## Alternative Method

If `pdftotext` was not available, we could also try `strings`:

```bash
strings challenge.pdf | grep -i "CIT"
```

or dump the extracted text to a file first:

```bash
pdftotext challenge.pdf output.txt
cat output.txt | grep -i "CIT"
```

This would reveal the same flag.

## Why This Works

The PDF did not require cracking, decoding, or heavy forensic carving. The important clue was that the document was formatted like an email. Email headers such as `FROM`, `TO`, `CC`, and `Subject` are easy to overlook because they look like normal document structure.

In this case, the `CC` field directly contained the flag.

## Flag

```text
CIT{m0j0_eng4g3d}
```
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
