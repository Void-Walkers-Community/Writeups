# DawgCTF Writeup – crazy.txt

## :pushpin: Challenge Description
A long, repetitive rant-like text filled with variations of the classic “crazy” chant, increasingly distorted with leetspeak and symbols.

Flag format: `DawgCTF{...}`

---

## :mag: Initial Analysis

At first glance, the file appears to be nonsense:

> "crazy? i was crazy once! they locked me in a room..."

This paragraph is repeated many times, but each repetition becomes more corrupted:
- Letters are replaced with numbers (`a → 4`, `e → 3`, `i → 1`, `o → 0`)
- Later replaced with symbols (`|`, `/\/\`, `(_)`, etc.)

This suggests **intentional obfuscation**, not randomness.

---

## :jigsaw: Key Observation

Each repetition ends with a **short fragment**, for example:
}pl
eh_
dne
s_e
sae
lp_
efi
l_y
m_f
o_l


These fragments differ across repetitions and appear structured.

---

## :brain: Strategy

Instead of focusing on the repeated paragraph, extract only the **ending fragments** from each block.

---

## :arrows_counterclockwise: Decoding Process

### Step 1: Collect Fragments

From each repeated block, take the trailing piece:
}pl, eh_, dne, s_e, sae, lp_, efi, l_y, m_f, o_l, ort, noc, _ll, a_t, sol, ev, ah, i{F, ...


---

### Step 2: Reverse the Order

The fragments are arranged **backwards**, so reverse the full list.

---

### Step 3: Reverse Each Fragment

Each fragment is also individually reversed:

| Fragment | Reversed |
|----------|----------|
| `efi`    | `ife`    |
| `lp_`    | `_pl`    |
| `dne`    | `end`    |

---

### Step 4: Reconstruct the Message

After reversing both:
- the **order of fragments**
- and **each fragment itself**

We get:

---

## :triangular_flag_on_post: Final Flag

---

## :performing_arts: Notes

- The repeated chant is **intentional noise**
- Increasing obfuscation is meant to **distract and overwhelm**
- The real signal is hidden in the **consistent structure at the end of each block**

---

## :test_tube: Takeaway

When faced with chaotic or repetitive data:
- Look for **patterns across repetitions**
- Focus on **what changes slightly each time**
- Extract small anomalies—they often form the hidden message

---

## :checkered_flag: Conclusion

This challenge is a classic example of:
> hiding a clear signal inside overwhelming noise

Once the pattern is identified, the solution becomes straightforward.


---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
