# HBKG Writeup

## Challenge

We are given a service with two menu options:

1. Rotate a point on the unit sphere, up to two times
2. Encrypt using a key derived from the tangent vector at a chosen point

The relevant code is in [HBKG.sage](/home/parth/HACK/crypto_hbkg/HBKG.sage:1).

## Source Analysis

The service generates:

- a hidden rotation matrix `R`
- a hidden axis vector `axis`

The rotation is a standard 3D rotation around `axis`.

For encryption, the service computes:

```python
t = get_tangent(v, axis)   # v x axis
k = t * random.randint(2^127, 2^128)
data = ','.join([str(c) for c in k])
h = hashlib.sha256(data.encode()).digest()
```

Then it uses `h` as the AES-ECB key.

So the whole problem is: can we choose a point `v` such that `v x axis = 0`?

Yes. If we submit `v = axis` (or `-axis`), then the cross product is zero, so:

```python
t = (0, 0, 0)
k = (0, 0, 0)
data = "0,0,0"
key = sha256(b"0,0,0")
```

That makes the AES key fully known.

## Recovering the Axis

We do not know `axis`, but we do have a rotation oracle twice.

Let:

- `p = (1,0,0)`
- `q = (0,1,0)`

and query the service for `R(p)` and `R(q)`.

For any rotation around axis `a`, the vectors:

- `p - R(p)`
- `q - R(q)`

both lie in the plane perpendicular to `a`.

Therefore their cross product is parallel to the rotation axis:

```text
a ~ (p - R(p)) x (q - R(q))
```

Normalizing gives the unit axis.

In the recovered instance, this simplifies nicely to:

```text
axis = (
  -sin(phi)*cos(theta),
  -sin(phi)*sin(theta),
   cos(phi)
)
```

with the exact rational-angle values coming from the service output.

## Why This Works

The challenge title hint is `HB`, which points to the Hairy Ball Theorem.

The code computes a tangent vector field on the sphere:

```python
get_tangent(point, axis) = point x axis
```

A tangent vector field on the sphere must vanish somewhere. In this construction, it vanishes exactly at the poles of the axis, namely `point = +-axis`.

So once we recover the axis, we can force the tangent vector to be zero and obtain a deterministic AES key.

## Exploit Script

```python
import socket
import select
import time
import re
import hashlib
import sympy as sp
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

HOST = "hbkg.ctf.theromanxpl0.it"
PORT = 9094

def recv_some(s, timeout=0.2):
    out = b""
    end = time.time() + timeout
    while time.time() < end:
        r, _, _ = select.select([s], [], [], max(0, end - time.time()))
        if not r:
            break
        chunk = s.recv(65536)
        if not chunk:
            break
        out += chunk
    return out.decode(errors="replace")

def recv_until(s, marker, timeout=10):
    data = ""
    end = time.time() + timeout
    while marker not in data and time.time() < end:
        chunk = recv_some(s, 0.5)
        if chunk:
            data += chunk
    return data

def sendline(s, msg):
    if isinstance(msg, str):
        msg = msg.encode()
    s.sendall(msg + b"\n")

def parse_last_tuple(text):
    tuples = re.findall(r"\([^\n]*\)", text)
    return sp.sympify(tuples[-1].replace("^", "**"))

s = socket.create_connection((HOST, PORT), timeout=10)
recv_until(s, "> ")

sendline(s, "1")
recv_until(s, "x: ")
sendline(s, "1")
recv_until(s, "y: ")
sendline(s, "0")
recv_until(s, "z: ")
sendline(s, "0")
q1 = sp.Matrix(parse_last_tuple(recv_until(s, "> ")))

sendline(s, "1")
recv_until(s, "x: ")
sendline(s, "0")
recv_until(s, "y: ")
sendline(s, "1")
recv_until(s, "z: ")
sendline(s, "0")
q2 = sp.Matrix(parse_last_tuple(recv_until(s, "> ")))

e1 = sp.Matrix([1, 0, 0])
e2 = sp.Matrix([0, 1, 0])
axis = (e1 - q1).cross(e2 - q2)
axis = sp.simplify(axis / sp.sqrt(axis.dot(axis)))
coords = [sp.sstr(sp.simplify(c)) for c in axis]

sendline(s, "2")
recv_until(s, "x: ")
sendline(s, coords[0])
recv_until(s, "y: ")
sendline(s, coords[1])
recv_until(s, "z: ")
sendline(s, coords[2])
text = recv_until(s, "\n", timeout=5) + recv_some(s, 1)

ct = bytes.fromhex(re.search(r"([0-9a-fA-F]{32,})", text).group(1))
key = hashlib.sha256(b"0,0,0").digest()
pt = unpad(AES.new(key, AES.MODE_ECB).decrypt(ct), AES.block_size)

print(pt.decode())
```

## Flag

```text
TRX{y0u_c4n7_c0mb_TheHairyCoconut_https://en.wikipedia.org/wiki/Hairy_ball_theorem}
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
