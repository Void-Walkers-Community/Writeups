# Vanilla raw

- **Category:** Forensics
- **Points:** 946
- **Artifact:** `memory.rar`
- **Flag format:** `KubSTU{...}`

## Summary

The provided `memory.raw` was not a usable RAM image in the usual sense. After extraction, it turned out to be a **2 GiB file filled almost entirely with zeroes**, with only one small non-zero region.  
The flag was hidden inside that region using **4-byte interleaving**: taking every 4th byte from the extracted payload (`blob[::4]`) reveals the flag as plaintext.

## 1. Extract the archive

`7z` could list the archive but could not unpack it because of the RAR5 method, so `unrar` was used:

```bash
mkdir -p vanilla_raw
cd vanilla_raw
wget -O memory.rar 'https://kubstu-ctf.online/files/a91e3888b1c16aac23f6aa300db03d71/memory.rar?token=...'
unrar x -o+ memory.rar
```

This produced:

- `memory.raw` — size `2147483648` bytes

## 2. Check whether the dump is real

Quick inspection showed the file starts and ends with zeroes:

```bash
xxd -l 256 memory.raw
xxd -s -256 -l 256 memory.raw
```

A coarse scan over 1 MiB chunks showed that only **one chunk** contains non-zero data:

```python
from pathlib import Path

p = Path("memory.raw")
chunk = 1024 * 1024
hits = []

with p.open("rb") as f:
    off = 0
    while True:
        data = f.read(chunk)
        if not data:
            break
        if any(data):
            hits.append(off)
        off += len(data)

print(hits)
```

Output:

```text
[1039138816]
```

Then the exact non-zero boundaries were located:

```text
start = 1039149037
end   = 1039151240
size  = 2204 bytes
```

## 3. Extract the only meaningful payload

```python
from pathlib import Path

p = Path("memory.raw")
start = 1039149037
end = 1039151240

with p.open("rb") as f:
    f.seek(start)
    blob = f.read(end - start + 1)

Path("nonzero.bin").write_bytes(blob)
print(len(blob))
```

Output:

```text
2204
```

At this point the challenge clearly was not about Volatility/profile detection.  
The payload had to be interpreted differently.

## 4. Deinterleave the payload

Testing byte-lane extraction showed that the first stream with stride 4 contains readable data:

```python
from pathlib import Path

blob = Path("nonzero.bin").read_bytes()
lane0 = blob[::4]

idx = lane0.find(b"KubSTU{")
print(idx)
print(lane0[idx:idx + 80])
```

Output:

```text
256
b'KubSTU{m3m0ry_unl1nk3d_tmpfs_f0r3ns1cs}...'
```

So the flag is:

```text
KubSTU{m3m0ry_unl1nk3d_tmpfs_f0r3ns1cs}
```

## Flag

```text
KubSTU{m3m0ry_unl1nk3d_tmpfs_f0r3ns1cs}
```
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

