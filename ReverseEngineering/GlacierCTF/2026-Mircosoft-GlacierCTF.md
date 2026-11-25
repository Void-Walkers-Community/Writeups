# mircosoft — Challenge Writeup

**Flag:** `gctf{m3m0ry_dump_DRM_c3rt_f0rg3d_7b2e}`

## TL;DR

A Windows PE loader (`mircosoft.exe`) decrypts and runs an embedded PE in
memory. The embedded PE is a DRM-style license checker that reads
`license.bin`, derives an XTEA key from the host machine's fingerprint
(CPUID brand string, C: volume serial, `MachineGuid` from the registry,
SMBIOS Type-1 UUID, and the first network adapter's MAC), and uses that
key to XTEA-CTR-decrypt a 64-byte flag stored inside the license.

The accompanying `admin_machine.dmp` minidump captures exactly the
fingerprint fields the binary queries, so the same key can be re-derived
offline and used to decrypt `license.bin` into the flag — i.e. the
"DRM cert" was forged using the memory dump.

## Files in the challenge

| File                | Role                                                       |
| ------------------- | ---------------------------------------------------------- |
| `mircosoft.exe`     | Outer loader (control-flow-flattened, no IAT, direct NT syscalls) |
| `license.bin`       | 104-byte license (`MSLC` + version + payload)              |
| `admin_machine.dmp` | Windows minidump of the admin's machine                    |
| `sha256sum`         | Checksums (verified OK)                                    |

## Stage 1 — Unpacking the outer loader

`mircosoft.exe` is a PE32+ with no imports directory and a state-machine
dispatcher (`ebx` is the state register, hashed with a MurmurHash-style
mixer on each transition). It resolves `kernel32.dll` exports by DJB2
hash (multiplier `0x21`, seed `0x1505`), walks the export table, and
locates `VirtualAlloc`, `LoadLibraryA`, `VirtualProtect`,
`GetProcAddress`, `ExitProcess`.

The 0x4800-byte payload starting at `.rdata+0` is the encrypted embedded
PE. The loader decrypts it via three steps (in order):

1. **Block permutation** — 72 blocks of 256 bytes are shuffled using a
   PCG LCG (`mult=0x5851f42d4c957f2d`, `inc=0x14057b7ef767814f`,
   seed `0xdead1337cafe4269`) driving a Fisher–Yates shuffle.
2. **Per-byte rotation + subtraction** — for each byte `i`:
   `data[i] = ROR(data[i], key2[i%32] & 7) - key1[i%32]`, where the
   32-byte `key1` (subtract) and `key2` (shift) tables are stored at
   `.rdata+0x4810` and `.rdata+0x4830`, XOR-decoded with
   `(i*0x5a + 0x3c) & 0xff`.
3. **Chained XOR stream cipher** — `out[i] = in[i] ^ key3[i%16] ^ prev`,
   `prev = in[i] + key3[(i+3)%16]`, with the 16-byte `key3` at
   `.rdata+0x4800`.

After decryption the result is a clean PE (`MZ...PE\0\0`).

## Stage 2 — Embedded PE: anti-analysis

The embedded PE has no IAT either — it resolves 16 `ntdll` syscalls by
DJB2 hash and stores their numbers at `.data+0x500`. It also installs an
**instrumentation callback** (`r15d=0x13371337` is the marker) and uses
two intentional exceptions to drive control flow:

- `int 0x2d` (BREAKPOINT) — runs the volume + `MachineGuid` collector.
- `ud2` (ILLEGAL_INSTRUCTION) — runs the SMBIOS + MAC collector and
  finalises the XTEA key.

## Stage 3 — Fingerprint → XTEA key

The fingerprint is built in `.data+0x70` and a custom MD5-style
compression function (`0x140001d00`) folds it into a 16-byte state at
`.data+0x4e0`:

```
for each 4-byte chunk of input:
    state[0] ^= chunk
    XTEA_encrypt(state[0..8],  key1 = .data+0x00)
    XTEA_encrypt(state[8..16], key2 = .data+0x10)
    state[3] ^= state[1]
    state[0] ^= state[2]
```

The XTEA here is **non-standard**: `v0 += (sum + key[sum&3]) ^ (v1 + ((v1<<4)^(v1>>5)))`
— the addition is outermost, not the XOR. Also note the state starts at
**BSS zero**, not at the MD5 IV constants — those constants are only
used by the separate filename-validation MD5 (`0x140001df0`), which was
the bug that took the longest to spot.

Three MD5 calls accumulate the state, in order:

| # | Source                            | Length | Data                                                |
| - | --------------------------------- | -----: | --------------------------------------------------- |
| 1 | CPUID leaves `0x80000002..4`      |     48 | `"Intel(R) Core(TM) Ultra 7 155H"` + zero padding  |
| 2 | Volume + `MachineGuid`            |     40 | C: volume serial + registry `MachineGuid` ASCII    |
| 3 | SMBIOS UUID + MAC                 |     22 | Type-1 UUID (16 B) + first adapter MAC (6 B)        |

Finally a 64-byte shellcode (NOT-decoded from `.rdata+0x4850`, then
XOR'd with `0x5a`) runs one round of a 128-bit ARX cipher
(`ROL(x,7) ^ next + ...`) over the state, and the result is copied to
`.data+0x480` — the 16-byte XTEA key.

### Fingerprint values recovered from `admin_machine.dmp`

| Field                | Where in the dump                | Value                                              |
| -------------------- | -------------------------------- | -------------------------------------------------- |
| CPUID brand          | seg 0 (`KUSER_SHARED_DATA`)      | `Intel(R) Core(TM) Ultra 7 155H`                   |
| C: volume serial     | seg 4 (`FILE_FS_VOLUME_INFORMATION` at +0x40) | `0xd48d4c91`                           |
| `MachineGuid`        | seg 1 (registry `SOFTWARE`)      | `658c05ff-e49e-40aa-b041-9a5f56886dea`             |
| SMBIOS Type-1 UUID   | seg 2 (raw SMBIOS, struct 1 +8)  | `3c31a857b2b700488ce185385968c8a8`                 |
| MAC of first adapter | seg 3 (`IP_ADAPTER_INFO` +0x198) | `52540036e664`                                     |

## Stage 4 — License format and decryption

`license.bin` (104 bytes):

```
[ 0:  4] "MSLC"           magic
[ 4:  8] 1                version
[ 8: 24] MD5 of filename  16-byte check (validates key was set up)
[24: 32] counter init     8-byte XTEA-CTR nonce (r14, r15)
[32: 96] ciphertext       64-byte XTEA-CTR-encrypted flag
[96:104] hash check       8-byte = custom_hash(license[0:96], key)
```

The custom hash (function `0x140001f90`) is XTEA in a CBC-like
compression: start `v0=v1=0`, for each 8-byte block XOR it into the
state and XTEA-encrypt, then one extra XTEA round at the end. Output is
`v0||v1` (8 bytes). This gives the validation oracle used to confirm
the derived key.

The CTR decryption (function `0x140001e60`) is 8 blocks of 8 bytes:
`flag[i] = XTEA_encrypt(r14+i, r15, key) ^ cipher[i]`.

## Recovered key and flag

- **XTEA key:** `3b8652afcde7de87294a383867f1a9f7`
- **Flag:** `gctf{m3m0ry_dump_DRM_c3rt_f0rg3d_7b2e}`

The flag theme ("memory-dump DRM cert forged") matches the attack
exactly: the admin's machine fingerprint was captured in the minidump,
the per-machine XTEA key was re-derived offline, and the license's
encrypted flag block was decrypted without ever running the binary on
the original hardware.

## Deliverables in `download/`

- [flag.txt](/home/z/my-project/download/flag.txt) — the flag
- [xtea_key.bin](/home/z/my-project/download/xtea_key.bin) — the 16-byte XTEA key
- [embedded_pe.bin](/home/z/my-project/download/embedded_pe.bin) — the decrypted inner PE
- [writeup.md](/home/z/my-project/download/writeup.md) — this writeup



