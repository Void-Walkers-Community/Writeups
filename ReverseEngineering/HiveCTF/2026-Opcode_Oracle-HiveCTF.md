# Opcode Oracle — Writeup

## Challenge

We are given a stripped artifact, not a normal executable. The file only contains metadata plus raw hex dumps for `.text`, `.rodata`, and `.bss`:

```text
arch: x86_64
format: raw section hex from a stripped Linux ELF
text_vaddr: 0x401000
rodata_vaddr: 0x402000
bss_vaddr: 0x403160
entry_vaddr: 0x401000
```

The goal is to recover the one input accepted by the hidden validator.

Final flag:

```text
HiveCTF{0pc0d3_0r4cl3_15_n0t_4_hum4n}
```

---

## 1. First look

The artifact gives raw bytes instead of an ELF with symbols. The important parts are:

- `.text` at `0x401000`
- `.rodata` at `0x402000`
- `.bss` at `0x403160`
- entry point at `0x401000`

Inside `.rodata`, there is an obvious fake flag:

```text
HiveCTF{ai_got_baited_for_real}
```

This is bait. Submitting it fails.

---

## 2. Rebuilding the layout mentally

Since the challenge gives virtual addresses, the raw bytes can be disassembled as x86-64 code starting at `0x401000`.

A simple way to inspect it is to extract the `TEXT` bytes and disassemble them with the correct base address.

Example workflow:

```bash
# Extract only the bytes from the TEXT section into text.hex manually or with a parser
xxd -r -p text.hex text.bin
objdump -D -b binary -m i386:x86-64 --adjust-vma=0x401000 text.bin
```

The program does roughly this:

```c
write(1, "whisper> ", 9);
read(0, input, 0x60);
strip_newline(input);

if (!check_0()) fail;
if (!check_1()) fail;
...
if (!check_7()) fail;

print("accepted\n");
```

The validator calls 8 small check functions through a jump table in `.rodata`.

---

## 3. Basic format check

One early function checks the input length and the flag wrapper.

The length check is:

```asm
cmp rax, 0x25
```

So the full input must be `0x25 = 37` bytes long.

The wrapper is checked with small reversible byte operations instead of direct comparisons. Reversing those checks gives:

```text
HiveCTF{............................}
```

The inner part has 28 characters.

It also checks that every inner character is one of:

```text
0-9, a-z, _
```

So the target shape is:

```text
HiveCTF{xxxxxxxxxxxxxxxxxxxxxxxxxxxx}
```

where `x` is lowercase, digit, or underscore.

---

## 4. The real checks

The rest of the validator uses arrays in `.rodata` and performs arithmetic/XOR checks over the 28 inner bytes.

Some examples of the check types:

### Permutation + XOR/add transform

One function permutes the 28 input bytes using an index table, mixes them with two byte arrays, and compares the result against a target array.

The core operation looks like:

```c
out[i] = ((input[perm[i]] ^ key1[i]) + key2[i]) & 0xff;
```

### Group sums

Another function splits the inner flag into 7 groups of 4 bytes and checks each group sum against 16-bit constants.

```c
input[4*i] + input[4*i+1] + input[4*i+2] + input[4*i+3] == target_sum[i]
```

### Group XORs

Another function checks XORs of every 4-byte group.

```c
input[4*i] ^ input[4*i+1] ^ input[4*i+2] ^ input[4*i+3] == target_xor[i]
```

### Table-based arithmetic

Other functions use small index arrays to select bytes, perform rotations/additions/XORs, and compare against target bytes.

The important observation is that all constraints are over the 28 inner characters only, and each character is limited to a small alphabet.

---

## 5. Solving approach

Because the alphabet is small:

```python
alphabet = b"abcdefghijklmnopqrstuvwxyz0123456789_"
```

we can solve the checks either by:

1. translating the assembly into Python constraints, or
2. using Z3, or
3. brute-forcing grouped constraints once the 4-byte group checks are identified.

The fastest path is to lift the checks into Python and search against the allowed alphabet.

---

## 6. Solver skeleton

This is the style of solver used: parse the known checks from `.rodata`, model the validator, and test candidate characters.

```python
#!/usr/bin/env python3

alphabet = b"abcdefghijklmnopqrstuvwxyz0123456789_"

# Final inner flag recovered from reversing the validator constraints.
inner = b"0pc0d3_0r4cl3_15_n0t_4_hum4n"
flag = b"HiveCTF{" + inner + b"}"

assert len(flag) == 0x25
assert len(inner) == 28
assert all(c in alphabet for c in inner)

print(flag.decode())
```

For a full automated solve, each assembly check can be copied into Python as a function returning `True` or `False`. The correct candidate is the one that passes all 8 functions.

---

## Flag

```text
HiveCTF{0pc0d3_0r4cl3_15_n0t_4_hum4n}
```
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
