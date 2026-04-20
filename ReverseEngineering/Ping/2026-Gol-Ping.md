# Challenge Writeup

**Author:** lexu  
**Flag format:** `ping{.*}`

## Initial Recon

cf.md
2 KB
# Gorbino's Quest of Life — Writeup

## Challenge Summary

We are given an ELF binary (`task`) and told the flag:

writeup.md
6 KB
Universal [CODE],  — Yesterday at 3:01 PM
https://ctf.knping.pl/teams/invite?code=eyJpZCI6NSwidiI6IjQ3NmYxNmFhYWMxYTU2MDIxMWQxNWUyN2Y0MjJjMmY2ZWRkNmE4MzAifQ.aeSg2Q.vE_JoHP8a6XGwzpw4qgXkqvRMHg
@Saber
Universal [CODE],  — Yesterday at 4:28 PM
@Null Horizon

:advaith_anim:  PingCTF has officially ended kudos to Parth0xu , parachyut, Saber and our MVP KingPinzz :thumbs_up: .. Good game team.. ggs. :letter: 

:p_: :i_: :n_: :g_:
Cutieeepieeee — Yesterday at 5:28 PM
The web chall for the ping for me is style
the first one i think i got the flag unintedeed just fire up simple payload of xss to fetch /flag and check on preview on id and it display the flag
The second one for me intrrating because its use prng check to guest the id number because it randomly changed if we refresh so need to create a python script and set up thw xss payload to bypass csp from app.js and setup the webhook to get the flag
Saber — Yesterday at 5:33 PM
@Null Horizon please share all the writeups
Cutieeepieeee — Yesterday at 5:41 PM
im not sure how to share my solution the file exceed size limit🤣
Saber — Yesterday at 5:42 PM
umm in github ?
you can upload them and share me the link
Saber — Yesterday at 5:57 PM
@Universal compile the writeups once 
like if anyone havent shared theirs then tell me
Universal [CODE],  — Yesterday at 6:05 PM
@Null Horizon everyone is requested to send all their write-ups of the respective solved challenges in this channel asap... Note: this text is for those you solve at least one challenge.
Universal [CODE],  — Yesterday at 7:16 PM
List:

@Saber  - sanity check
@karam - gol, ctf madness 
@parachyut - tuttis, loicense
KingPinzz - ping-notes, pingdrafts, ctf review.

@Saber these are the members and the list of challenges they solved.
Saber — Yesterday at 7:28 PM
Give me the rank card too
For the logs @Universal
Universal [CODE],  — Yesterday at 7:30 PM
Ok
Image
Saber — Yesterday at 7:45 PM
List:
@karam - gol(R) , ctf madness(R)  ⁠│null-horizon⁠
@parachyut - tuttis(P), loicense(R)  ---> Shared
@Cutieeepieeee  - ping-notes(w), pingdrafts(w), ctf review.(W)

@Saber these are the members and the list of challenges they solved. 
@parachyut @Cutieeepieeee please provide the writeups within 6 hours from now
Void
APP
 — Yesterday at 10:21 PM
Added Null Supreme to abhas87_48823.
Saber — Yesterday at 10:22 PM
@parachyut @Cutieeepieeee please share your writeups
Saber — Yesterday at 11:07 PM
@Cutieeepieeee @parachyut guys
need the writeups
Saber — 8:59 AM
@Cutieeepieeee @parachyut please buddy give the writeup
Saber — 9:24 AM
@Cutieeepieeee only yours writeups left 😭
Saber — 9:52 AM
@Cutieeepieeee broo
﻿
# Gorbino's Quest of Life — Writeup

## Challenge Summary

We are given an ELF binary (`task`) and told the flag:

- matches `ping_.*`
- is shorter than 64 characters (including prefix)

The binary is a custom cellular-automaton-like simulation over a `64 x 48` grid, with very obfuscated symbol names and heavy multithreading.

Recovered flag:

`ping_500hoursofmindglorbingaction`

---

## 1. Initial Recon

```bash
file /media/sf_kali/task
nm -n /media/sf_kali/task
strings -n 6 /media/sf_kali/task | head -n 80
```

Notable findings:

- non-stripped PIE ELF
- useful symbols exist (`TheStrongDecideTheNatureOfSin`, `BlueHouseGTechExec`, etc.)
- large embedded ASCII grids in `.rodata`
- function `main` reads one string via `scanf`, then calls the checker

---

## 2. Static Disassembly Highlights

Key functions:

- `main` -> reads input -> calls `TheStrongDecideTheNatureOfSin` -> prints success/fail
- `AGoodPartyRequiresABloodSacrifice`:
  - converts input bytes into **bitplanes** (8 planes)
  - returns a 64x48 matrix-like object
- `BlueHouseGTechExec`:
  - per-cell worker thread
  - updates one cell over many generations using neighborhood bits and lookup table `GAME`
- embedded constants:
  - `GAME` at `0x3100`: 1024 chars (`'0'/'1'`) => transition lookup
  - `KRILLYOUSELF` at `0x3500`: 3072 chars (`' '`/`'#'`) => one target grid
  - `TheOnePieceIsReal` at `0x4100`: 3072 chars (`' '`/`'#'`) => another target grid

---

## 3. Runtime Strategy

Direct brute force is infeasible:

- simulation count is `0x537a` generations
- binary spawns huge numbers of threads

So instead we extract **exact runtime graph wiring** once and mathematically invert generations.

### 3.1 Run under loader (environment-compatible)

```bash
cp /media/sf_kali/task /tmp/gorbino_task
chmod +x /tmp/gorbino_task
```

Use loader explicitly:

```bash
/lib64/ld-linux-x86-64.so.2 /tmp/gorbino_task
```

### 3.2 Break before thread launch and snapshot heap

In `gdb`:

```gdb
set pagination off
set disable-randomization on
set stop-on-solib-events 1
starti
c
# wait until /tmp/gorbino_task is mapped
info proc mappings
# text base from mapping: e.g. 0x7ffff7fb6000 for file offset 0x1000
# break at TheStrong...+0x526 (start of thread creation loop)
b *0x7ffff7fb6a86
set stop-on-solib-events 0
c
# provide any short input when prompted, e.g. A
```

At breakpoint, dump relevant heap range:

```gdb
dump binary memory /tmp/heap_snapshot.bin 0x7ffff7fff000 0x7ffff82f8000
quit
```

---

## 4. Inversion Model

At the breakpoint:

- each cell object is size `0x98`
- each cell contains:
  - incoming channels count at `+0x80`
  - outgoing channels count at `+0x84`
  - target bits at `+0x89` and `+0x8a` (for final two generations)
- the neighborhood order is encoded by pointer wiring (incoming channel list order)

Thus we can reconstruct exact directed neighborhood mapping for every cell.

Transition per cell is effectively:

- gather incoming bits into accumulator `acc` (shift+or in configured order)
- `f = GAME[2*((cur<<8)|acc)] - '0'`
- reverse relation from worker logic:
  - `prev = f XOR next`

Given `s_T` and `s_{T-1}` from `+0x89/+0x8a`, iterate backward `T=0x537a` times to recover `s_0`.

Then decode first 8 rows as bitplanes (LSB-first per character column), producing up to 64 chars.

---

## 5. Solver Script (core)

```python
import struct, pathlib, numpy as np

# rodata
ro = pathlib.Path('/tmp/rodata.bin').read_bytes()
rbase = 0x3000
GAME = (np.frombuffer(ro[0x3100-rbase:0x3100-rbase+1024], dtype=np.uint8) - 48).astype(np.uint8)

W, H = 64, 48
N = W * H
STEPS = 0x537a

# heap snapshot
base = 0x7ffff7fff000
mem = pathlib.Path('/tmp/heap_snapshot.bin').read_bytes()

def q(addr):
    return struct.unpack_from('<Q', mem, addr - base)[0]
def u32(addr):
    return struct.unpack_from('<I', mem, addr - base)[0]
def u8(addr):
    return mem[addr - base]

# pointer to column-array from this run
arr = 0x7ffff80007b0
cols = [q(arr + i*8) for i in range(W)]

cell_addr = []
index = {}
for x in range(W):
    for y in range(H):
        a = cols[x] + y*0x98
        index[a] = x*H + y
        cell_addr.append(a)
cell_addr = np.array(cell_addr, dtype=np.uint64)

# channel -> source cell map (from outgoing lists)
src = {}
for a in cell_addr:
    outc = u32(int(a)+0x84)
    for i in range(outc):
        ch = q(int(a)+0x40+i*8)
        src[ch] = index[int(a)]

# incoming neighbor table in exact runtime order
maxd = 8
nb = np.zeros((N, maxd), dtype=np.int32)
mask = np.zeros((N, maxd), dtype=np.uint8)
for a in cell_addr:
    i = index[int(a)]
    inc = u32(int(a)+0x80)
    for k in range(inc):
        ch = q(int(a)+k*8)
        nb[i, k] = src[ch]
        mask[i, k] = 1

# final states from checker memory
s_next = np.array([u8(int(a)+0x89) for a in cell_addr], dtype=np.uint8)  # s_T
s_cur  = np.array([u8(int(a)+0x8a) for a in cell_addr], dtype=np.uint8)  # s_{T-1}

# reverse time
for _ in range(STEPS, 0, -1):
    acc = np.zeros(N, dtype=np.uint16)
    for k in range(maxd):
        vals = s_cur[nb[:,k]]
        m = mask[:,k]
        acc = np.where(m, (acc << 1) | vals, acc)
    f = GAME[2*((s_cur.astype(np.uint16) << 8) | acc)]
    prev = (f ^ s_next).astype(np.uint8)
    s_next, s_cur = s_cur, prev

# decode s_0 first 8 rows as bitplanes -> ASCII
s0 = s_next.reshape(W, H)
flag = ''.join(chr(int(np.sum((s0[x,:8].astype(np.uint16)) << np.arange(8, dtype=np.uint16)))) for x in range(W))
flag = flag.split('\x00', 1)[0]
print(flag)
```

Output:

`ping_500hoursofmindglorbingaction`

---

## 6. Final Flag

`ping_500hoursofmindglorbingaction`
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
