## CTF Writeup: Looking Down

### Challenge Overview
The challenge presents a series of character strings that do not follow traditional cryptographic patterns.

* **Challenge String:** `1qazs3edc 456rfvbngh 7ujmil0p;/`
* **Flag Format:** `UDCTF{}`

---

### Initial Analysis
The input is divided into three distinct groups containing a mix of letters, numbers, and symbols. Standard decryption methods like Caesar ciphers or Base64 decoding do not yield results. 

The title **"Looking Down"** serves as the vital hint.

### Key Insight
The phrase "Looking Down" refers to the physical act of looking down at a **QWERTY keyboard**. Instead of interpreting the characters as text, they must be treated as coordinates on a keyboard grid to visualize the shapes they form.

### Step 1: Keyboard Mapping
To solve this, we visualize the standard QWERTY layout:

* **Top row:** 1 2 3 4 5 6 7 8 9 0
* **Q row:** q w e r t y u i o p
* **A row:** a s d f g h j k l ;
* **Z row:** z x c v b n m , . /

---

### Step 2: Plotting the Groups

#### Group 1: 1qazs3edc
Mapping these keys on the grid:
```
1 . 3
q . e
a s d
z . c
```
The perimeter of these keys forms the shape of the letter **H**.

#### Group 2: 456rfvbngh
Mapping these keys on the grid:
```
4 5 6
r . .
f g h
v b n
```
The path traced by these keys forms the shape of the letter **E**.

#### Group 3: 7ujmil0p;/
Mapping these keys on the grid:
```
7 . . 0
u i . p
j . l ;
m . . /
```
The arrangement of these keys forms the shape of the letter **N**.

---

### Final Result
Combining the visualized letters from each group spells out: **HEN**.

**Final Flag:**
`UDCTF{HEN}`
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
