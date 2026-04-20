# Writeup

**Flag:** `ping{01_m8_U_g07_4_l01c3n53_f0r_h4ck1ng?}`

## TL;DR

This challenge looks like it wants you to reverse some weird signature scheme, but the real bug is much simpler: the "license key" format lets us choose the final equation in a `10 x 10` linear system.

The HWID gives us 9 equations.  
The first 11 bytes of the key give us the 10th equation.  
The rest of the key is just 10 float values, encoded as hex.

So instead of fighting the binary's fake-complex verifier, we can:

1. Build the 9 equations from the HWID.
2. Pick 11 metadata bytes for the last row.
3. Solve the linear system directly.
4. Encode the 10 float32 results into the key.
5. Retry metadata until the residual is tiny.

That was enough to solve the remote checker 100 times and get the flag.

## First Look

The zip contains:

- `license_checker.py`
- `SmartAttend`

The checker is actually super useful, because it tells us what the server expects:

```python
from generate_valid_pairs import Pair, validate_pair

for _ in range(100):
    hwid = "".join(random.choice(HWID_DIGITS) for _ in range(99))
    print(f"{hwid}")
    key_input = input()

    pair = Pair(hwid=hwid, key_bytes=key_input)
    if not validate_pair(pair):
        good = False
        break
```

So the task is not "find one magic key."  
It's "write a keygen that works for arbitrary 99-character HWIDs."

That immediately tells us we need to understand the actual validation logic, not just patch the program.

## The Binary Tries Very Hard To Waste Your Time

The ELF is not stripped, which is honestly the nicest thing this challenge does for us. A quick look with `nm -C` / `dwarfdump` gives away the real moving parts:

- `parse_hwid_to_planes`
- `derive_last_plane_from_key_fields`
- `decode_signature_from_key`
- `verify_signature`
- `probe_signature_candidate`
- `compute_system_loss`

There are also a bunch of fake-looking helper names, base64 blobs, "solver tips," and compliance notices. Some of the hints are real, but a lot of the binary is there to make the target feel more complicated than it is.

Once I ignored the menu fluff and followed the activation path, the shape of the challenge got a lot cleaner.

## What The HWID Actually Becomes

There is a global:

```cpp
float planes[10][11];
```

That is the whole game.

`parse_hwid_to_planes` takes the 99-byte HWID and splits it into 9 chunks of 11 characters each.

For each chunk:

- the first 10 characters become coefficients
- the 11th character becomes the constant term

So each chunk becomes one linear equation in 10 unknowns.

That means the 99-byte HWID gives us **9 equations** total.

If we write one chunk as:

```text
c0 c1 c2 c3 c4 c5 c6 c7 c8 c9 rhs
```

then the equation is:

```text
ord(c0)*x0 + ord(c1)*x1 + ... + ord(c9)*x9 = ord(rhs)
```

After 9 chunks, we have 9 such equations.

## What The Key Format Actually Is

The key has two parts.

### Part 1: 11 bytes of metadata

This part is used by `derive_last_plane_from_key_fields`.

The DWARF info gives away the variable names:

- `no_of_users`
- `validity_days`
- `license_no`
- `version`

That sounds fancy, but in practice it's just 11 raw bytes:

- bytes `0..9` become the last row's coefficients
- byte `10` becomes the last row's constant term

So the metadata gives us the **10th equation**.

### Part 2: 80 bytes of hex

`decode_signature_from_key` parses the rest with `%8x`, 10 times.

So after the first 11 bytes, the binary expects:

- 10 chunks
- each chunk is 8 hex characters
- each chunk is reinterpreted as an IEEE-754 float32

That gives the candidate signature:

```text
x0, x1, x2, ..., x9
```

So the full key format is:

```text
[11 metadata chars][10 * 8 hex chars]
```

Total length:

```text
11 + 80 = 91 characters
```

## The Important Realization

At this point, the verifier stops looking like crypto and starts looking like algebra.

We already get 9 equations from the HWID.  
The key lets us choose the 10th equation ourselves.

That means we can build a full `10 x 10` system:

```text
A * x = b
```

where:

- the first 9 rows come from the HWID
- the last row comes from our chosen metadata
- `x` is the 10-float signature we need to encode

So instead of "guessing" a signature, we can just solve the system.

That is the whole exploit.

## But What About `verify_signature`?

This is the part that tries to scare you.

`verify_signature` does not simply check exact equality. It:

- parses the 10 float values from the key
- also tries the negated version
- runs `probe_signature_candidate`
- computes loss / residual / drift
- compares against a few thresholds

That sounds much worse than it is.

The reason is that `probe_signature_candidate` is still evaluating the same linear system. It just wraps the check in a little optimization routine and then enforces some tolerances.

The useful constants from the DWARF data were:

- `probe_iterations = 64`
- `probe_gradient_tolerance = 0.001`
- `probe_residual_eps = 0.015`
- `max_initial_loss = 4.0`
- `max_final_loss = 0.01`
- `max_cumulative_loss = 4.0`
- `max_drift_norm_sq = 0.04`

Those tolerances are loose enough that a normal direct solve works fine, as long as we produce a float32-friendly solution.

So the plan becomes:

1. Choose metadata.
2. Solve the system in double precision.
3. Cast the solution to float32.
4. Check the float32 residual.
5. If the residual is still tiny, use it.
6. Otherwise try different metadata.

No emulation, no symbolic execution, no patching needed.

## Building The Keygen

The solver I ended up using was just Gaussian elimination with partial pivoting.

For each remote HWID:

1. Split it into 9 rows of 11 chars.
2. Convert chars to `ord(...)`.
3. Randomly generate 11 metadata chars.
4. Use those bytes as the 10th row.
5. Solve for 10 unknowns.
6. Convert the solution to float32.
7. Encode each float as an 8-digit uppercase hex word.
8. Return:

```text
metadata + encoded_float_words
```

I looped until I found a row whose float32 residual was very small. In practice this was reliable.

The final script is here:

[smartattend_keygen.py](/Users/achyut/Downloads/smartattend_keygen.py)

## Why This Works

The checker is effectively trusting user-controlled bytes to define part of the system that the signature is supposed to satisfy.

That means the "license key" is not proving knowledge of some secret.  
It is literally allowed to help define the equations being checked.

So we are not breaking a signature scheme.  
We are solving a system that the program mistakenly lets us finish writing.

That's why the whole thing collapses into a keygen.

## Solving The Remote

The remote service prints a banner, then 100 HWIDs.  
For each HWID, we send back a generated 91-character key.

After 100 successful rounds:

```text
Impressive
ping{01_m8_U_g07_4_l01c3n53_f0r_h4ck1ng?}
```

## Final Flag

```text
ping{01_m8_U_g07_4_l01c3n53_f0r_h4ck1ng?}
```

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
