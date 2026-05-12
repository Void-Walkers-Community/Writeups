## PNG is a Lie — Part 1: Extracting the Image

### Challenge

We're given a file `weird_file.thc` with an unknown format.

---

### Step 1 — Identify the encoding

Opening the file reveals it's full of 👍 and 👎 emojis scattered among random alphanumeric noise like `hOv`, `DgIsm`, `Vq`, etc. The key observation is that only the emojis carry information — the text is a **red herring**.

The two emojis map naturally to binary:

* 👍 = `1`
* 👎 = `0`

---

### Step 2 — Verify it's byte-aligned

Counting the emojis:

* **👍:** 2,848,721
* **👎:** 2,832,927
* **Total:** 5,681,648

$$5,681,648 \div 8 = 710,206$$

The result is exactly **710,206** with no remainder. This confirms the data is cleanly byte-aligned and intentional.

---

### Step 3 — Decode

To extract the data, follow these steps:

1. Extract only the emojis in order.
2. Map them to their corresponding bits.
3. Group the bits into 8-bit chunks.
4. Convert those chunks into bytes.

---

### Step 4 — Identify the file

The first 8 bytes of the output are `\x89 P N G \r \n \x1a \n`, which is the **PNG magic header**. This confirms the file is a valid **1000×1000 RGB PNG** image.



---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


