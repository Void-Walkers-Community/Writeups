# The Ancient Note - Writeup

## Challenge
We are given `ancient_note.txt`, described as an ancient philosophical manuscript.
Flag format is provided as:

`KubSTU{...}`

## Initial Observations
At first glance, the text looks normal English prose, but two things stand out:

1. Strange character rendering in some words.
2. Suspicion of steganography due to the themed hint about “hidden truth” and “spaces between”.

## Analysis
### 1. Inspect file metadata
The file is UTF-8 with CRLF line endings, which allows hidden Unicode characters.

### 2. Reveal hidden characters
By printing the text with line numbers and then enumerating non-ASCII code points, we find many:

- `U+200B` (Zero Width Space)
- `U+200C` (Zero Width Non-Joiner)

These are invisible in normal viewing and are commonly used for binary steganography.

### 3. Convert zero-width chars to bits
Map:

- `U+200B` -> `0`
- `U+200C` -> `1`

Collecting these in order yields a binary stream:

`01001011011101010110001001010011010101000101010101111011...`

### 4. Decode as ASCII
Grouping into 8-bit bytes and converting to ASCII gives:

`KubSTU{h1dd3n_truth_b3tw33n}`

This perfectly matches the expected flag format.

## Flag
`KubSTU{h1dd3n_truth_b3tw33n}`


---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
