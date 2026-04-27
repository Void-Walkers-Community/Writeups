# Seven Minutes Writeup

## Challenge

- Name: `Seven Minutes`
- Category: `Crypto`
- Points: `436`
- Author: `thek0der`
- Files:
  - `chall.py`
- Remote:
  - `nc 74934548253bcab8.ctf.ac.upt.ro 9019`

## Summary

The provided Python file is not an encryption oracle or a signature service. It is a tiny quaternion-LWE generator:

```python
q = 65537
n = 15
m = 17
B = 16
```

It samples:

- a random matrix `A` of size `17 x 15` over quaternions mod `65537`
- a secret vector `s` of length `15`, where every quaternion coefficient lies in `[-16, 16]`
- an error vector `e` of length `17`, with the same small bound

and returns:

```python
b = A * s + e
```

The remote service prints one instance `(A, b)` and asks us to send back `s`.

So the task is a bounded-distance decoding problem in a very low-dimensional LWE-like system. The intended solve is to linearize the quaternion multiplication into ordinary integer matrices and then recover the planted short vector with lattice reduction.

## Step 1: Linearize quaternion multiplication

For a fixed quaternion:

```text
u = (a, b, c, d)
```

and an unknown quaternion:

```text
x = (x0, x1, x2, x3)
```

left multiplication `u * x` is linear in the coordinates of `x`:

```text
u * x = L(u) · x
```

with:

```text
L(a,b,c,d) =
|  a  -b  -c  -d |
|  b   a  -d   c |
|  c   d   a  -b |
|  d  -c   b   a |
```

Each quaternion equation therefore becomes `4` scalar equations. Since the original system has:

- `17` equations
- `15` secret quaternions

we obtain an ordinary modular linear system:

```text
M · s_vec = b_vec + e_vec   (mod q)
```

where:

- `M` is `68 x 60`
- `s_vec` is the `60`-dimensional flattened secret
- `e_vec` is the `68`-dimensional flattened error
- every entry of `s_vec` and `e_vec` lies in `[-16, 16]`

That size is small enough for a direct lattice attack.

## Step 2: Build a Kannan-style embedding

We want integers `s_vec`, `z`, and `e_vec` such that:

```text
M · s_vec - q · z - b_vec = e_vec
```

with both `s_vec` and `e_vec` tiny.

Construct the column basis:

```text
      | I   0   0 |
C  =  | M -qI -b |
      | 0   0  mu|
```

where:

- `I` is the `60 x 60` identity matrix
- `-qI` is the `68 x 68` diagonal block
- `mu` is a small embedding constant

For the correct secret and carry vector `z`, the lattice contains:

```text
C · (s_vec, z, 1)^T = (s_vec, e_vec, mu)
```

This vector is extremely short because:

- every secret coefficient is at most `16`
- every error coefficient is at most `16`
- `mu` can be chosen small, for example `200`

So the problem reduces to finding that short vector.

## Step 3: Recover the short vector with lattice reduction

I used `fpylll` with:

- `LLL`
- then `BKZ-30`

on the `129`-dimensional embedding lattice.

After reduction, one basis vector has:

- last coordinate `±200`
- first `60` coordinates all in `[-16, 16]`
- next `68` coordinates also in `[-16, 16]`

That vector is exactly:

```text
(s_vec, e_vec, mu)
```

up to sign.

A final verification step checks that the recovered `s_vec` really satisfies:

```text
M · s_vec - b_vec = e_vec   (mod q)
```

with the balanced representatives of `e_vec`.

Once that passes, reshape `s_vec` back into `15` quaternions and send it as JSON.

## Step 4: Solve the remote service

Using the recovered secret against the live instance returns:

```text
Correct! Flag: CTFAC{383e7754330895c91f33095fd0665c83babd9841886e4f3d61194ec0c69d68b7}
```

## Minimal Solver

The full reusable solver is saved alongside this writeup as:

- `ac26/seven-minutes-solve.py`

It:

1. connects to the remote service
2. parses the JSON instance
3. linearizes the quaternion system
4. builds the embedding lattice
5. runs `LLL` and `BKZ`
6. extracts the short vector
7. sends the recovered secret back to the server

Run it with the virtualenv that has `fpylll` installed:

```bash
/home/priyanshu/ac26/.venv/bin/python /home/priyanshu/ac26/seven-minutes-solve.py
```

## Why this works

This challenge looks unusual because it is written over quaternions, but cryptanalytically it is still just a noisy linear system over a prime field with a very small secret and a very small error.

The quaternion layer does not add meaningful hardness here:

- multiplication by a known quaternion is linear
- the whole system flattens cleanly into a standard modular linear problem
- the secret and error are both so small that a lattice embedding isolates the planted vector

At this dimension, BKZ finds the solution comfortably.

## Flag

```text
CTFAC{383e7754330895c91f33095fd0665c83babd9841886e4f3d61194ec0c69d68b7}
```

## Takeaway

The core mistake is parameter selection. A scheme that resembles LWE only inherits LWE-like security if the dimensions and noise are large enough to resist lattice attacks.

Here:

- the flattened secret dimension is only `60`
- the error dimension is only `68`
- both secret and error are bounded by `16`

That makes the planted vector far too short relative to the embedding lattice, so straightforward lattice reduction recovers it directly.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
