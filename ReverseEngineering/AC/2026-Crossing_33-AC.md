# Crossing 33 Writeup

## Challenge

The attachment set contains:

- `crossing-33.exe`
- `flag.txt`

The binary is a 32-bit Windows PE, while `flag.txt` is not plaintext at all. A quick look at the executable strings already reveals the overall intent:

```text
usage: %s <input-plaintext> <output-flag.txt>
failed to derive round keys through the gate
failed to pack plaintext
```

So this is not a flag checker. It is a packer: the program takes plaintext, derives round keys through an internal "gate", encrypts the result into a custom container, and writes that container out as `flag.txt`.

That means the clean solve path is to reverse the packer and run it backwards on the provided output file.

## Summary

`flag.txt` starts with a custom `C33GATE` header:

```text
00000000: 43 33 33 47 41 54 45 00 01 00 00 00 30 00 00 00
00000010: 48 00 00 00 00 00 00 00 f2 74 6d 28 f3 61 ff b1
00000020: 24 86 63 e8 ab aa e8 01 ab 9e 09 28 00 00 00 00
```

The useful fields are:

- magic: `C33GATE\\x00`
- version: `1`
- data offset: `0x30`
- plaintext length: `0x48` (`72`)
- four 32-bit seed values at `0x18..0x27`
- expected 32-bit FNV-1a of the plaintext at `0x28..0x2b`

The executable contains:

- a key-scheduler routine at `0x4015b0..0x40181e`
- four 12-entry qword tables at:
  - `0x40a440`
  - `0x40a4a0`
  - `0x40a500`
  - `0x40a560`

Rather than manually lifting the scheduler from assembly, the simplest reliable approach is:

1. parse the header
2. emulate the scheduler with the four header seeds to recover the 12 round keys
3. reverse the 12-round block transform for each 16-byte ciphertext block
4. truncate the recovered plaintext to the header length
5. recompute FNV-1a and compare it to the header value

That produces the flag:

```text
CTFAC{ddbd780508840afbe4a64c7f76498e3a5b0ba0853f0a679f223705670f626fae}
```

## Step 1: Parse the container header

The provided `flag.txt` is a small binary container, not an actual text file. The solver treats the layout as:

```text
0x00  8 bytes   magic        = b"C33GATE\\x00"
0x08  4 bytes   version      = 1
0x0c  4 bytes   data offset  = 0x30
0x10  4 bytes   length       = 0x48
0x14  4 bytes   header field mixed into the cipher
0x18  4 bytes   seed0
0x1c  4 bytes   seed1
0x20  4 bytes   seed2
0x24  4 bytes   seed3
0x28  4 bytes   expected FNV-1a
0x30  ...       encrypted payload
```

For the supplied sample:

- `length = 0x48`
- `seed0 = 0x286d74f2`
- `seed1 = 0xb1ff61f3`
- `seed2 = 0xe8638624`
- `seed3 = 0x01e8aaab`
- `expected FNV = 0x28099eab`

Those fields are enough to drive both the key schedule and the final integrity check.

## Step 2: Recover the round keys through the gate

The main obstacle is the internal scheduler. The executable derives 12 qword round keys from the four 32-bit seeds, but reimplementing that logic directly from the assembly is unnecessary work.

The solver instead uses:

- `pefile` to map the PE image into memory
- `unicorn` to emulate the scheduler function in place

The emulation starts at `0x4015b0`, stops at `0x40181e`, seeds the expected registers and stack slots, and reads the resulting 96-byte output buffer as 12 little-endian qwords.

This is the key step that makes the solve reproducible without reconstructing every helper primitive by hand.

## Step 3: Reverse the 12-round block transform

Once the round keys are known, the rest of the format is a straight block-by-block decryption.

The executable also embeds four static qword tables, each with 12 entries. For every 16-byte block, the solver:

1. xors the two ciphertext halves with rotated seed material
2. runs the 12 rounds in reverse order
3. recomputes the per-round mixing values from:
   - the recovered round key
   - the qword tables
   - the block index
   - a 64-bit constant derived from the header field at `0x14` and the plaintext length
4. removes the final whitening layer based on the seeds and block counter

The block transform is easiest to recognize as a custom Feistel-like network over two 64-bit halves with a heavily mixed round function. Reversing it directly in Python is simpler than trying to coerce the Windows executable into decrypting for us.

## Step 4: Validate the recovered plaintext

After decryption, the solver truncates the output to the header length and recomputes a standard 32-bit FNV-1a:

```python
fnv = 0x811C9DC5
for byte in plaintext:
    fnv ^= byte
    fnv = (fnv * 0x1000193) & 0xFFFFFFFF
```

For the provided file, the recomputed digest matches the header value `0x28099eab`, which confirms the plaintext was recovered correctly.

The recovered plaintext is:

```text
CTFAC{ddbd780508840afbe4a64c7f76498e3a5b0ba0853f0a679f223705670f626fae}
```

## Flag

```text
CTFAC{ddbd780508840afbe4a64c7f76498e3a5b0ba0853f0a679f223705670f626fae}
```

## Reproduction

A reproducible solver is saved in `crossing33/solve.py`.

In this workspace, the bundled virtualenv already contains the required dependencies, so the solve reproduces exactly with:

```bash
/home/priyanshu/crossing33/.venv/bin/python \
  /home/priyanshu/crossing33/solve.py \
  /home/priyanshu/crossing33/crossing-33.exe \
  /home/priyanshu/crossing33/flag.txt
```

Output:

```text
CTFAC{ddbd780508840afbe4a64c7f76498e3a5b0ba0853f0a679f223705670f626fae}
```

The solver also verifies the recovered plaintext against the container's header FNV before printing the result.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
