# Break The Chain - Writeup

## Summary
We are given:
- A protocol client: `/media/sf_kali/client.py`
- A manual: `/media/sf_kali/SST-documentation.pdf`
- Remote service: `nc 4.178.152.74 9000`

Goal: trigger drone self-destruction and recover flag `THC{...}`.

---

## 1. Initial Recon

From `client.py`, communication format is:
- `4-byte big-endian length`
- followed by raw payload bytes

The client receives one payload, then sends it back.

So this is a tampering challenge: modify ciphertext in-transit and relay it back.

---

## 2. Manual Analysis

From the PDF:
- Action codes:
  - `1`: Self-destruction (autodelete)
  - `2`: Attack
  - `3`: Next target
  - `4`: Hide
- Packet format per action (`16 bytes`, big-endian):
  - `id` (`uint16`) -> 2 bytes
  - `robot_id` (`uint16`) -> 2 bytes
  - `action_type` (`uint8`) -> 1 byte
  - `timestamp` (`uint64`) -> 8 bytes
  - `pad` -> 3 bytes

Whole payload:
- `count` (`uint32`) + `count * 16-byte actions`

Important offset:
- First action starts at byte `4` of payload
- `action_type` for first action is at payload offset `4 + 2 + 2 = 8`

---

## 3. Oracle Behavior

When sending untouched data, server prints inferred decrypted actions, e.g.:
- Attack / Next target / Hide

This leak confirms:
- We can alter decrypted command meaning by flipping ciphertext bytes.
- The service accepts modified payload as long as framing stays valid.

---

## 4. Exploit Idea

Assume stream-like XOR behavior on `action_type` byte (or equivalent malleability):
- `cipher = plain XOR key`
- To force plain from `{2,3,4}` to `1`, try XOR deltas:
  - `2 -> 1` needs delta `0x03`
  - `3 -> 1` needs delta `0x02`
  - `4 -> 1` needs delta `0x05`

So we only need to flip byte `payload[8]` with one of `{3,2,5}`.

---

## 5. Final Trigger

Using original packet length + payload, only modify one byte:
- `payload[8] ^= 3` (or try `2`, then `5` if needed)

Successful response included:
- `___Autodelete___`
- `THC{4lL_Dr0Nz-R-g0N3}`

---

## 6. Minimal Solver

```python
#!/usr/bin/env python3
import socket
import struct

HOST = "4.178.152.74"
PORT = 9000

def recv_exact(sock, n):
    b = b""
    while len(b) < n:
        c = sock.recv(n - len(b))
        if not c:
            raise ConnectionError("closed")
        b += c
    return b

with socket.create_connection((HOST, PORT)) as s:
    l = struct.unpack(">I", recv_exact(s, 4))[0]
    payload = bytearray(recv_exact(s, l))

    for delta in (3, 2, 5):
        mod = bytearray(payload)
        mod[8] ^= delta
        s2 = socket.create_connection((HOST, PORT))
        l2 = struct.unpack(">I", recv_exact(s2, 4))[0]
        p2 = bytearray(recv_exact(s2, l2))
        p2[8] ^= delta
        s2.sendall(struct.pack(">I", len(p2)) + p2)
        out = b""
        while True:
            c = s2.recv(4096)
            if not c:
                break
            out += c
        txt = out.decode(errors="ignore")
        print(txt)
        s2.close()
        if "THC{" in txt:
            break
```

---

## Flag

`THC{4lL_Dr0Nz-R-g0N3}`


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
