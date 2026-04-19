# Transport Trouble Writeup

## Challenge Description

We are given a cargo shipment challenge involving fake HoneySig signing specifications.

The goal is to:

1. Find the real HoneySig specification.
2. Implement the algorithm described by the real spec.
3. Verify which cargo flag has the authentic Queen Bee signature.

---

## Files

After downloading and extracting the cargo archive, the important files are:

```text
spec_*.txt
flags_with_honeysig.json
secret.txt
queen_public_key.pem
```

The `spec_*.txt` files contain different signing algorithms, but only one of them is authentic.

The JSON file contains cargo entries with flags and signatures.

---

## Step 1: Identify the Real Specification

Each spec file is signed using HoneySig. Since we are given the Queen Bee's public key, we can verify which spec has a valid signature.

After checking the specs, the valid one is:

```text
spec_06.txt
```

The real specification says:

```text
1) XOR with 0x8eef00d55f83
2) reverse, by bytes
3) append the secret string
4) calculate the SHA256 hash
```

So the correct HoneySig algorithm is:

```text
flag
-> XOR with repeating key 8e ef 00 d5 5f 83
-> reverse bytes
-> append secret.txt content
-> SHA256
```

---

## Step 2: Understand the Signing Algorithm

The XOR key is:

```text
0x8eef00d55f83
```

Converted to bytes:

```python
key = bytes.fromhex("8eef00d55f83")
```

For each byte of the flag, we XOR it with the corresponding byte of the key.

Since the key is shorter than the flag, it repeats:

```python
xored = bytes(c ^ key[i % len(key)] for i, c in enumerate(flag_bytes))
```

Then the result is reversed:

```python
rev = xored[::-1]
```

Then the secret string is appended:

```python
rev + secret
```

Finally, SHA256 is calculated:

```python
hashlib.sha256(rev + secret).hexdigest()
```

---

## Step 3: Verify the Cargo Flags

We compare our generated HoneySig hash against each cargo signature in `flags_with_honeysig.json`.

If the calculated hash matches the provided signature, that cargo is legitimate.

---

## Solver Script

```python
import json
import hashlib

key = bytes.fromhex("8eef00d55f83")

with open("secret.txt", "rb") as f:
    secret = f.read()

with open("flags_with_honeysig.json") as f:
    cargos = json.load(f)

def honeysig(flag):
    flag_bytes = flag.encode()

    # Step 1: XOR with repeating key
    xored = bytes(
        b ^ key[i % len(key)]
        for i, b in enumerate(flag_bytes)
    )

    # Step 2: Reverse by bytes
    reversed_bytes = xored[::-1]

    # Step 3 + 4: Append secret and SHA256
    return hashlib.sha256(reversed_bytes + secret).hexdigest()

for cargo in cargos:
    flag = cargo["flag"]
    signature = cargo["signature"]

    if honeysig(flag) == signature:
        print("[+] Valid cargo found!")
        print(flag)
        break
```

---

## Output

Running the solver gives:

```text
[+] Valid cargo found!
HiveCTF{cb34a3e5bc9d5b2ad9afa5b42b5d83cd}
```

---

## Flag

```text
HiveCTF{cb34a3e5bc9d5b2ad9afa5b42b5d83cd}
```

---

## Key Takeaway

This challenge is about separating the real signing specification from many fake ones.

Once the valid spec is found using the Queen Bee's public key, the rest is straightforward:

```text
XOR -> reverse -> append secret -> SHA256
```

The cargo whose signature matches this algorithm is the real one.



---

* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


