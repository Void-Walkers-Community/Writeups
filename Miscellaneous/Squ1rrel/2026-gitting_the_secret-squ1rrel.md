# Gitting the Secret — Writeup

## Challenge Description

"I'm a supremely talented developer who would never ever commit secrets to git. You'll never find the flag, let alone all three parts of it!"

---

## Approach Overview

This challenge focuses on Git forensics — recovering hidden or deleted data from a repository.

Initial observations:
- `git log` showed no commits
- `.git/` directory contained objects, logs, and a suspicious `secret/` folder

This indicates detached or hidden Git history.

---

## Step 1 — Find Hidden Objects

Command:
git fsck --lost-found

This revealed:
- Dangling commits
- Dangling blobs

Example output:
dangling commit 9d219e...
dangling blob e0de69...

These are unreferenced objects that may contain deleted data.

---

## Step 2 — Recover Checkpoint 1

Inspect commit:
git cat-file -p 9d219e026839a10ba01f792cf26c79a3a44cbd7d

Get tree:
git cat-file -p 213c65d35cc63c05dc0384440bfaca271a52db51

Found:
flag_1.txt → blob 920984...

Extract:
git cat-file -p 920984763899e54c82db401ec6d9db7b5540754a

Part 1:
4WpKZIx9qnhWDQ7L1MTTfMgLzSL2dj

---

## Step 3 — Extract from Dangling Blobs

Dump blobs:
git cat-file -p <blob> | strings

From one blob:
BR43O1z6Oh4uZB9

Part 2:
BR43O1z6Oh4uZB9

---

## Step 4 — Hidden Packfile

Inside `.git/secret/`:
knapsack.pack

This is a Git packfile containing hidden objects.

---

## Step 5 — Unpack Packfile

git unpack-objects < knapsack.pack
git fsck --lost-found

New commit appeared:
dangling commit 2ef0d8...

---

## Step 6 — Recover Checkpoint 3

git cat-file -p 2ef0d8...

Tree:
git cat-file -p 51fef7...

Found:
flag_3.txt → blob 93bbb5...

Extract:
git cat-file -p 93bbb5c17dea12d25aedf03b8996935a5fc950ba

Part 3:
2kp2hO0KjST5nlsWu72RXIddAovYpsebEiUvSJgjfAX8MvwFpwz9uheyD

---

## Step 7 — Identify Encoding

From index.html:
Home base: 62

This indicates Base62 encoding.

---

## Step 8 — Decode Each Part

Python script:

import string

alphabet = string.digits + string.ascii_uppercase + string.ascii_lowercase

def base62_decode(s):
    num = 0
    for char in s:
        num = num * 62 + alphabet.index(char)
    return num

def decode_part(s):
    num = base62_decode(s)
    return num.to_bytes((num.bit_length() + 7)//8, 'big').decode()

parts = [
    "4WpKZIx9qnhWDQ7L1MTTfMgLzSL2dj",
    "BR43O1z6Oh4uZB9",
    "2kp2hO0KjST5nlsWu72RXIddAovYpsebEiUvSJgjfAX8MvwFpwz9uheyD"
]

for p in parts:
    print(decode_part(p))

---

## Final Flag

squ1rrel{d0nut_c0mM1T_uR_s3cR3ts_w1tH_g1T_12b7160d77d8fbd071f42e0cbccad934}
