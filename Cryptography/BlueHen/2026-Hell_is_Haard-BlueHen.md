## CTF Writeup: Hell is haard

### Challenge Overview
We are provided with a file containing a list of **2048 integers**. The goal is to decode this sequence to reveal the hidden flag.

* **Title:** "Hell is haard"
* **Flag Format:** `udctf{...}`

---

### Step 1: Analyzing the Clues
Two major hints point toward the solution:
1.  **The Title:** The misspelling of "hard" as "**haard**" is a direct nod to the **Haar Wavelet Transform**.
2.  **Data Size:** The list contains exactly **2048** numbers. Since $2048 = 2^{11}$, this power-of-two length is a standard requirement for signal processing transforms like the Haar transform.

### Step 2: Applying the Inverse Haar Transform
Because the provided data represents a transformed signal, we must apply the **Inverse Haar Wavelet Transform** to reconstruct the original values. This process transforms the frequency-domain integers back into a spatial-domain signal, which in this case, represents ASCII values.

### Step 3: Data Segmentation
After performing the inverse transform, we obtain 2048 values. These are divided into **four distinct blocks**, each containing 512 characters.

### Step 4: Decoding the Shifts
Each block uses a different Caesar-style shift (normalization) that must be reversed to obtain readable text:

| Block | Encoding / Shift | Correction Strategy |
| :--- | :--- | :--- |
| **0** | Plain ASCII | No change needed |
| **1** | Shifted by $+91$ | Subtract 91 from each value |
| **2** | Shifted by $-211 \pmod{256}$ | Add 211 $\pmod{256}$ |
| **3** | Shifted by $-118 \pmod{256}$ | Add 118 $\pmod{256}$ |

---

### Step 5: Flag Recovery
Once the shifts are normalized, the resulting text reveals a passage from **Dante’s Inferno**. Embedded within the literary text is the following string:

> `This is udctf{h3lp_1t5_c0ld_d0wn_h3r3}`

**Final Flag:**
`udctf{h3lp_1t5_c0ld_d0wn_h3r3}`
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
