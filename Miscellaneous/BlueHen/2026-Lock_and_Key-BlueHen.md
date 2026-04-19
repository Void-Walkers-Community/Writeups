## CTF Writeup: Lock and Key

### Challenge Overview
The challenge provides a scrambled string that maintains a clear flag-like structure.

* **Scrambled String:** `GEJYU?<dro{Yflcbi+`
* **Flag Format:** `UTCTF{}`

---

### Initial Analysis
The string clearly mirrors the expected flag format: `?????{????????}`. Because the structure (the placement of braces) is preserved, it is unlikely that the solution involves complex block ciphers like AES or RSA.

The title **"Lock and Key"** serves as the primary clue. In a CTF context, "key" often points toward:
* Physical keyboard layouts.
* Key-based ciphers (like Vigenère).
* Input device mapping.

### Observation
The presence of symbols such as `<`, `?`, and `+` is a strong indicator of keyboard output. This suggests the challenge involves a **keyboard layout mismatch**, where the text was typed on one layout but interpreted as another.

### Key Insight
The most common layout swap used in these challenges is between **QWERTY** and **Dvorak**.

* **The Theory:** The flag was typed while the computer was set to the **Dvorak** layout, but the output was recorded as if it were **QWERTY**.

---

### Execution: Dvorak to QWERTY
By mapping the characters from their positions on a Dvorak keyboard back to their corresponding keys on a QWERTY keyboard, we perform the following conversion:

* **Input:** `GEJYU?<dro{Yflcbi+`
* **Conversion:** Dvorak → QWERTY

### Result
Applying the mapping reveals the original intended text:

`UDCTF{Whos_Typing}`

**Final Flag:**
`UDCTF{Whos_Typing}`
---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
