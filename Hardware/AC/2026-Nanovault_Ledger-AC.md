# NanoVault Ledger Writeup

## Challenge

- Name: `Nanovault Ledger`
- Points: `100`
- Author: `thek0der`
- Service: `b2dd140336c9df86.ctf.ac.upt.ro:8728`

## Summary

The challenge ships a firmware image named `nanovault-ledger-v3.4.1.bin`. The image is a custom `NVFWIMG` container holding four sections:

- `kernel`
- `loader`
- `rootfs`
- `manifest`

Extracting the root filesystem reveals the real target, `/usr/sbin/nanovaultd`, which speaks a compact APDU-like protocol over stdin/stdout.

The intended break is a nonce-reuse bug in `INS 0x04`. Two signatures over different 64-bit messages reuse the same `r`, which is fatal for the daemon's Schnorr-like scheme. Once the signing secret is recovered, it becomes possible to forge the proof accepted by `INS 0x21`, which in turn returns the contents of `/flag.txt`.

One detail matters for local testing: a helper harness may inject a fake `/flag.txt` such as `CTFAC{local_nanovault_flag}`. That placeholder only proves the exploit path works. The real flag must be recovered from the remote service.

## Step 1: Extract the firmware image

A quick hex dump of the firmware header shows a simple custom package format:

```text
00000000: 4e 56 46 57 49 4d 47 00 01 00 00 00 04 00 00 00  NVFWIMG.........
00000010: 6b 65 72 6e 65 6c 00 00 00 00 00 00 00 00 00 00  kernel..........
00000020: b0 00 00 00 00 00 00 00 68 f9 15 01 00 00 00 00  ........h.......
00000030: 85 43 e6 81 00 00 00 00 6c 6f 61 64 65 72 00 00  .C......loader..
```

The container format is:

- magic: `NVFWIMG\\x00`
- version: `1`
- section count: `4`
- repeated section records containing:
  - a 16-byte ASCII name
  - a file offset
  - a length
  - a CRC32
  - flags

Parsing the header gives:

```text
kernel   @ 0x000000b0 size 0x115f968 crc ok
loader   @ 0x0115fa18 size 0x0310051 crc ok
rootfs   @ 0x0146fa69 size 0x016a000 crc ok
manifest @ 0x015d9a69 size 0x0000020 crc ok
```

The extracted root filesystem contains the challenge daemon:

```text
/usr/sbin/nanovaultd
```

## Step 2: Understand the APDU protocol

The daemon uses a very small framing layer:

```text
[2-byte big-endian length][CLA INS P1 P2 Lc][data]
```

The exploit only needs five instructions:

- `INS 0x01`: returns 24 bytes of public state
- `INS 0x20`: returns one 64-bit context value
- `INS 0x22`: returns another obfuscated 64-bit context value
- `INS 0x04`: signs an attacker-chosen 64-bit message
- `INS 0x21`: checks a proof and returns the flag on success

Talking to the local daemon yields:

```text
0x01 -> 434aab5a0b4559c76e312e3e591e89106624008295359d53
0x20 -> 11adcd2b32ef0873
0x22 -> 6701a7df93201ba0
0x04(0) -> a6ec958d21b674c90be6f1e63d11ffb2
0x04(1) -> d19e8dcdc7625ead08611b78547fa48e
0x04(2) -> d19e8dcdc7625ead0d62511fa89799c5
```

The critical observation is that the two signatures over messages `1` and `2` share the same first half:

```text
r1 = r2 = 0xd19e8dcdc7625ead
```

That is the entire vulnerability.

## Step 3: Recover the signing secret

The daemon implements a custom Schnorr-like construction over:

```text
P = 2^64 - 1487
Q = 2^60 - 93
G = 2^16
```

The challenge function for `INS 0x04` can be lifted directly from the binary and reproduced as:

```python
def chal_ins4(msg, r):
    first = mix(0xEB20D0EB15FB4EED ^ rol(msg, 53) ^ rol(r, 13))
    second = mix(0x9068758AFDA776F5 ^ first ^ r ^ msg)
    return (second % (Q - 1)) + 1
```

With nonce reuse, the signatures satisfy the standard relation:

```text
s1 - s2 = (c1 - c2) * x mod Q
```

So the secret key is:

```text
x = (s1 - s2) * (c1 - c2)^(-1) mod Q
```

Using the local transcript gives:

```text
x = 0x361e90d3a3e5c09
pub_x = 0x6624008295359d53
pow(G, x, P) = 0x6624008295359d53
```

The recovered `x` matches the advertised public key exactly, so the break is confirmed.

## Step 4: Forge the `INS 0x21` proof

Once `x` is known, the rest of the verification logic can be reconstructed from the daemon.

The solver derives:

```text
ctx50 = (mix(0x69BCA3B10553C63F ^ m0 ^ M1 ^ x) % (Q - 1)) + 1
ctx70 = ror(wire22, 19) ^ 0x753D9B48B9CA00AA ^ M1
q2    = mix(ctx50 ^ ctx60 ^ ctx70 ^ 0x4A0B7BDA61EAAE45)
h0    = mix(0xF6F5F2904627D708 ^ ctx60 ^ m2)
```

The proof accepted by `INS 0x21` is another Schnorr-style tuple. Choosing:

```text
r = 1
```

collapses the relation into a direct computation of:

```text
s = c * ctx50 mod Q
```

where `c` is the verifier challenge reproduced as:

```python
def chal_ins21(r, h0, h0_pre):
    pre_a = mix_pre(0x6801F42AF7ECB14E ^ rol(r, 13) ^ rol(h0, 53))
    tmp = (h0_pre ^ r ^ pre_a ^ ((h0_pre ^ pre_a) >> 31)) & MASK
    second = mix(0x00FA157BF658A734 ^ tmp)
    return (second % (Q - 1)) + 1
```

For the local instance, the forged values are:

```text
r  = 0x0000000000000001
s  = 0x0e95a4260fb54e96
q2 = 0x65b31f55a07517b1
```

Serialized payload:

```text
00000000000000010e95a4260fb54e9665b31f55a07517b1
```

Sending that as the body of `INS 0x21` causes the daemon to return `/flag.txt`.

## Step 5: Retrieve the real flag from the service

The local rootfs is useful for debugging, but it does not contain the real challenge flag. In many test setups, `/flag.txt` is overwritten with a placeholder value. The real solve must target the remote service.

Running the exploit against the challenge endpoint:

```bash
python3 solve_nanovault.py --host b2dd140336c9df86.ctf.ac.upt.ro --port 8728
```

returns:

```text
CTFAC{d0718944e902f030832e72164d737935fd286a7c7b23c09e2c8ab19e9c7daec3}
```

## Flag

```text
CTFAC{d0718944e902f030832e72164d737935fd286a7c7b23c09e2c8ab19e9c7daec3}
```

## Reproduction

The exploit is saved as `solve_nanovault.py`.

Local validation:

```bash
python3 solve_nanovault.py --local-rootfs /home/priyanshu/nanovault/rootfs_run
```

That only returns whatever `/flag.txt` currently contains in the local rootfs.

Real solve:

```bash
python3 solve_nanovault.py --host b2dd140336c9df86.ctf.ac.upt.ro --port 8728
```

On April 25, 2026, that returned:

```text
CTFAC{d0718944e902f030832e72164d737935fd286a7c7b23c09e2c8ab19e9c7daec3}
```
---
* [🔙 Back to Hardware Directory](../)
* [🔙 Back to Hardware Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
