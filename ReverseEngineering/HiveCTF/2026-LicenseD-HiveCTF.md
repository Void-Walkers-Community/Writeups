# LicenseD Writeup

**Challenge:** LicenseD  
**Category:** Reverse Engineering  
**Flag:** `HiveCTF{vm_unpack3d_4nd_cr4ck_th3_b3ast_0f_r3v3}`

---

## 1. Challenge Summary

The challenge gives only a stripped Linux ELF binary:

```bash
file licensed
```

Output:

```text
licensed: ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped
```

Running it shows that it expects exactly one argument:

```bash
./licensed
```

```text
usage: ./licensed <license-key>
```

The goal is to recover a valid license key.

---

## 2. First Look

Using `strings` gives an obvious-looking flag:

```bash
strings -a licensed
```

Interesting strings:

```text
usage: %s <license-key>
Access granted. Welcome.
Invalid license key.
HiveCTF{n0t_th3_r1ght_flag_k33p_d1gg1ng_d33p3r}
```

The `HiveCTF{n0t_th3_r1ght_flag_k33p_d1gg1ng_d33p3r}` string is bait.

```bash
./licensed 'HiveCTF{n0t_th3_r1ght_flag_k33p_d1gg1ng_d33p3r}'
```

```text
Invalid license key.
```

So the real key is not stored directly as a printable string.

---

## 3. Static Analysis

Disassembling the binary:

```bash
objdump -d -M intel licensed > disasm.txt
```

The main validation logic has three important parts:

1. It checks the license length.
2. It runs a custom bytecode/VM routine.
3. It applies a final byte transform and compares the result against a 48-byte encrypted target.

The length check is here:

```asm
10e3: call   strlen@plt
10e8: cmp    rax,0x30
10ec: je     ...
```

So the license key must be exactly:

```text
0x30 = 48 bytes
```

That matches the expected flag length.

---

## 4. Extracting the Target Bytes

The encrypted target is stored in `.rodata` at virtual address `0x20a0`.

Dump `.rodata`:

```bash
objdump -s -j .rodata licensed
```

Relevant bytes:

```text
20a0 de af 6a 72 f3 fd 4c 49 d7 61 a1 61 8a 9b 94 31
20b0 b6 bb f9 e9 f6 f0 50 66 76 b1 04 cc 16 f2 ef 22
20c0 47 fa 16 b2 c0 56 fb e1 c4 1c d1 5c 7a d9 8a f5
```

In Python form:

```python
target = bytes.fromhex(
    "deaf6a72f3fd4c49d761a1618a9b9431"
    "b6bbf9e9f6f0506676b104cc16f2ef22"
    "47fa16b2c056fbe1c41cd15c7ad98af5"
)
```

---

## 5. Understanding the VM

The binary contains a small virtual machine. The bytecode starts around `.rodata+0xe0`, or virtual address `0x20e0`.

The VM uses:

| Area | Purpose |
|---|---|
| `rsp+0x30` | 8 VM registers |
| `rsp+0x38` | VM memory, initially containing the user input |
| `rsp+0xb8` | VM stack |
| `.rodata+0x20e0` | bytecode program |

The dispatch loop reads one byte as an opcode:

```asm
1140: movzx eax, dx
1146: cmp BYTE PTR [rsi+rax], 0xd
114a: ja  11f3
1150: movzx eax, BYTE PTR [rsi+rax]
1154: movsxd rax, DWORD PTR [rdi+rax*4]
115b: jmp rax
```

This means opcodes `0x00` to `0x0d` are valid. Anything larger exits the VM.

From the handlers, the VM instruction set is roughly:

| Opcode | Operation |
|---:|---|
| `0` | halt |
| `1` | `reg[a] = imm` |
| `2` | `reg[a] = reg[b]` |
| `3` | `reg0 = reg[a] + reg[b]` |
| `4` | `reg0 = reg[a] ^ reg[b]` |
| `5` | `reg0 = rol(reg[a], imm)` |
| `6` | `reg0 = ror(reg[a], imm)` |
| `7` | `mem[imm] = reg[a]` |
| `8` | `reg[a] = mem[imm]` |
| `9` | `reg0 = reg[a] & reg[b]` |
| `10` | `reg0 = reg[a] - reg[b]` |
| `11` | `reg0 = reg[a] * imm` |
| `12` | `push reg[a]` |
| `13` | `pop reg[a]` |

The VM transforms the input into a 48-byte intermediate buffer at VM memory offset `0x40`.

---

## 6. Final Transform

After the VM finishes, the program copies 48 bytes from the VM output buffer and applies this final transform backwards from byte `47` to byte `0`:

```c
for (int i = 47; i >= 0; i--) {
    buf[i] = ror8(buf[(i + 7) % 48] ^ buf[i], (i % 3) + 1);
}
```

Then the transformed buffer is compared with the encrypted target bytes from `.rodata`.

So the solving strategy is:

1. Reverse the final transform to recover the VM output.
2. Emulate the VM symbolically / brute-force each byte relation.
3. Recover the original 48-byte license key.

---

## 7. Solver Script

Save as `solve.py`:

```python
#!/usr/bin/env python3

from pathlib import Path

BIN = Path("./licensed").read_bytes()

TARGET = bytes.fromhex(
    "deaf6a72f3fd4c49d761a1618a9b9431"
    "b6bbf9e9f6f0506676b104cc16f2ef22"
    "47fa16b2c056fbe1c41cd15c7ad98af5"
)

# Bytecode from .rodata: 0x20e0 until the final 0xff terminator.
BYTECODE = BIN[0x20e0:0x272b]


def rol(x, n):
    n &= 7
    return ((x << n) & 0xff) | (x >> (8 - n))


def ror(x, n):
    n &= 7
    return (x >> n) | ((x << (8 - n)) & 0xff)


def reverse_final_transform(target):
    """
    Invert:

        for i in range(47, -1, -1):
            buf[i] = ror(buf[(i + 7) % 48] ^ buf[i], (i % 3) + 1)

    For i <= 40, buf[i + 7] has already become final target[i + 7].
    For i >= 41, the index wraps around and uses original buf[0..6].
    """
    original = [0] * 48

    for i in range(40, -1, -1):
        k = (i % 3) + 1
        original[i] = rol(target[i], k) ^ target[i + 7]

    for i in range(47, 40, -1):
        k = (i % 3) + 1
        original[i] = rol(target[i], k) ^ original[i - 41]

    return bytes(original)


VM_EXPECTED_OUTPUT = reverse_final_transform(TARGET)


def emulate_vm(inp):
    regs = [0] * 8
    mem = [0] * 256
    stack = [0] * 32
    sp = 0
    pc = 0

    mem[:48] = inp

    while pc < len(BYTECODE):
        op = BYTECODE[pc]

        if op > 13:
            break

        if op == 0:
            break

        elif op == 1:
            # reg[a] = imm
            regs[BYTECODE[pc + 1] & 7] = BYTECODE[pc + 2]
            pc += 3

        elif op == 2:
            # reg[a] = reg[b]
            regs[BYTECODE[pc + 1] & 7] = regs[BYTECODE[pc + 2] & 7]
            pc += 3

        elif op == 3:
            # reg0 = reg[a] + reg[b]
            regs[0] = (regs[BYTECODE[pc + 1] & 7] + regs[BYTECODE[pc + 2] & 7]) & 0xff
            pc += 3

        elif op == 4:
            # reg0 = reg[a] ^ reg[b]
            regs[0] = regs[BYTECODE[pc + 1] & 7] ^ regs[BYTECODE[pc + 2] & 7]
            pc += 3

        elif op == 5:
            # reg0 = rol(reg[a], imm)
            regs[0] = rol(regs[BYTECODE[pc + 1] & 7], BYTECODE[pc + 2])
            pc += 3

        elif op == 6:
            # reg0 = ror(reg[a], imm)
            regs[0] = ror(regs[BYTECODE[pc + 1] & 7], BYTECODE[pc + 2])
            pc += 3

        elif op == 7:
            # mem[imm] = reg[a]
            mem[BYTECODE[pc + 1]] = regs[BYTECODE[pc + 2] & 7]
            pc += 3

        elif op == 8:
            # reg[a] = mem[imm]
            regs[BYTECODE[pc + 1] & 7] = mem[BYTECODE[pc + 2]]
            pc += 3

        elif op == 9:
            # reg0 = reg[a] & reg[b]
            regs[0] = regs[BYTECODE[pc + 1] & 7] & regs[BYTECODE[pc + 2] & 7]
            pc += 3

        elif op == 10:
            # reg0 = reg[a] - reg[b]
            regs[0] = (regs[BYTECODE[pc + 1] & 7] - regs[BYTECODE[pc + 2] & 7]) & 0xff
            pc += 3

        elif op == 11:
            # reg0 = reg[a] * imm
            regs[0] = (regs[BYTECODE[pc + 1] & 7] * BYTECODE[pc + 2]) & 0xff
            pc += 3

        elif op == 12:
            # push reg[a]
            if sp < len(stack):
                stack[sp] = regs[BYTECODE[pc + 1] & 7]
                sp += 1
            pc += 2

        elif op == 13:
            # pop reg[a]
            if sp > 0:
                sp -= 1
                regs[BYTECODE[pc + 1] & 7] = stack[sp]
            pc += 2

    # VM output buffer begins at VM memory offset 0x40.
    return bytes(mem[0x40:0x40 + 48])


def recover_flag():
    """
    The bytecode processes each input byte mostly independently.

    We recover the flag one position at a time:
    - start with a dummy 48-byte buffer
    - brute-force printable bytes for one position
    - emulate the VM
    - keep the byte that makes the corresponding VM output byte match
    """
    alphabet = (
        b"abcdefghijklmnopqrstuvwxyz"
        b"ABCDEFGHIJKLMNOPQRSTUVWXYZ"
        b"0123456789"
        b"_{}"
    )

    candidate = bytearray(b"A" * 48)

    # Known flag format helps reduce the search space.
    prefix = b"HiveCTF{"
    candidate[:len(prefix)] = prefix
    candidate[-1] = ord("}")

    for i in range(48):
        if i < len(prefix) or i == 47:
            continue

        for c in alphabet:
            test = candidate[:]
            test[i] = c
            out = emulate_vm(test)

            if out[i] == VM_EXPECTED_OUTPUT[i]:
                candidate[i] = c
                break
        else:
            raise RuntimeError(f"could not recover byte at index {i}")

    return bytes(candidate)


flag = recover_flag()
print(flag.decode())

assert emulate_vm(flag) == VM_EXPECTED_OUTPUT
```

Run it:

```bash
python3 solve.py
```

Output:

```text
HiveCTF{vm_unpack3d_4nd_cr4ck_th3_b3ast_0f_r3v3}
```

---

## 8. Verification

Submit the recovered key to the binary:

```bash
./licensed 'HiveCTF{vm_unpack3d_4nd_cr4ck_th3_b3ast_0f_r3v3}'
```

Output:

```text
Access granted. Welcome.
```

So the final flag is:

```text
HiveCTF{vm_unpack3d_4nd_cr4ck_th3_b3ast_0f_r3v3}
```

---

## 9. Key Takeaways

This challenge was not about finding a hidden string directly. The visible flag from `strings` was a decoy.

The real logic was:

```text
input license
    ↓
custom VM bytecode
    ↓
48-byte intermediate buffer
    ↓
final rotate/xor transform
    ↓
compare with encrypted target
```
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
