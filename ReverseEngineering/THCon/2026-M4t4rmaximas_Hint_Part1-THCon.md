# M4terM4xima's HINT (part 1/2)

## Challenge Context
We are given a tar archive containing:
- `INSTRUCTIONS.md`
- `HINT.elf`

The instructions mention this is a RISC-V binary and that **two flags** exist in the program.

## Initial Triage

### 1. File type
```bash
file HINT.elf
```
Result: `ELF 32-bit LSB executable, UCB RISC-V, statically linked, not stripped`

So this is a RISC-V 32-bit statically linked executable with symbols intact (nice for reversing).

### 2. Strings reconnaissance
```bash
strings -n 5 HINT.elf
```
Useful hits:
- `No HINT here`
- `Are you sure that you are looking for HINT?`
- `Congratulation, you just found a HINT`
- Symbol names like `maybe_HINT`, `main`, `_start_rust`

`maybe_HINT` immediately looked like the core logic.

## Static Reversing
Because native `objdump` lacked RISC-V disassembly support in this environment, I used `gdb-multiarch`:

```bash
gdb-multiarch -q -batch \
  -ex 'file HINT.elf' \
  -ex 'set architecture riscv:rv32' \
  -ex 'disassemble maybe_HINT'
```

### Key logic found in `maybe_HINT`
Inside `maybe_HINT`, the program:
1. Reads input bytes.
2. Applies a rolling XOR transform over the input buffer:
   - `buf[0] = buf[0] ^ 0x55`
   - For each `i > 0`: `buf[i] = buf[i] ^ old_buf[i-1]`
3. If length is exactly 20, compares transformed bytes against a hidden 20-byte constant at `0x80000ddc` via `memcmp`.

Hidden 20-byte constant (hex):
```
01 1c 0b 38 17 19 1c 49 5a 1f 17 1d 43 0c 4f 17 49 03 01 4e
```

## Recovering the Input (the Flag)
We invert the transform:
- `orig[0] = enc[0] ^ 0x55`
- `orig[i] = enc[i] ^ orig[i-1]`

Quick solver:
```python
enc = bytes.fromhex('011c0b3817191c495a1f171d430c4f174903014e')
out = []
out.append(enc[0] ^ 0x55)
for i in range(1, len(enc)):
    out.append(enc[i] ^ out[i-1])
print(bytes(out))
```

Decoded plaintext:
```
THC{lui zero, ox123}
```

## Flag (Part 1)
`THC{lui zero, ox123}`

## Notes
- This binary is intentionally noisy with Rust/runtime/trap code, but the important part is the local transform + memcmp in `maybe_HINT`.
- The challenge statement says there are two flags, so this is **part 1/2**.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
