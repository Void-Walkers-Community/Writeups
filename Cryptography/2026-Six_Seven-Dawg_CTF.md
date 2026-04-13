# Six Seven — Writeup

## Challenge

We are given the following Python code and ciphertext:

```python
from Crypto.Util.strxor import strxor
from secrets import flag
import os

assert flag[:8] == b"DawgCTF{" 

def gen(start):
    return (((6 * 7) * (start - 6) * 7) + ((start * 6) - 7) * (start ^ 6)) % 255

def encrypt(message):
    start = os.urandom(1)
    key = start
    for i in range(1, len(message)):
        key += gen(key[i-1]).to_bytes(1, "big")
    return strxor(key, message)

ct = encrypt(flag)
print("ct =", ct.hex())
```

Ciphertext:

```text
9f2eadbd998e9ca1aab6bfbba9bf85afa9bf85a9bfb9a8bfaea985b3b485a3b5afa885a9aea8bfbbb785b9b3aab2bfa8a985ece3b8bcbfebe3eebbbceee9bceab9bea7
```

## Observations

This is a **stream cipher** construction:

- A one-byte random `start` value is generated.
- A keystream is produced from that byte using `gen()`.
- The plaintext is encrypted with XOR:

```python
ciphertext = key XOR message
```

That means if we know part of the plaintext, we can recover the matching keystream bytes:

```text
key = ciphertext XOR plaintext
```

The code also tells us that the flag begins with:

```python
b"DawgCTF{"
```

So we have a known-plaintext attack immediately.

## Recovering the keystream

Let the ciphertext bytes be `ct[i]`.
Since:

```text
ct[i] = key[i] XOR flag[i]
```

we get:

```text
key[i] = ct[i] XOR flag[i]
```

Using the known prefix `DawgCTF{`, the first 8 keystream bytes are:

```text
key[0:8] = db 4f da da da da da da
```

Now check whether this agrees with the generator.

- `key[0] = 0xdb`
- `gen(0xdb) = 0x4f`
- `gen(0x4f) = 0xda`
- `gen(0xda) = 0xda`

So once the generator reaches `0xda`, it stays there forever. In other words, the keystream becomes:

```text
db 4f da da da da da da da da da da ...
```

That makes the rest of the ciphertext trivial to decrypt.

## Decryption script

```python
ct = bytes.fromhex("9f2eadbd998e9ca1aab6bfbba9bf85afa9bf85a9bfb9a8bfaea985b3b485a3b5afa885a9aea8bfbbb785b9b3aab2bfa8a985ece3b8bcbfebe3eebbbceee9bceab9bea7")

def gen(start):
    return (((6 * 7) * (start - 6) * 7) + ((start * 6) - 7) * (start ^ 6)) % 255

start = ct[0] ^ ord('D')  # known prefix: DawgCTF{
key = bytes([start])
for _ in range(1, len(ct)):
    key += bytes([gen(key[-1])])

pt = bytes(c ^ k for c, k in zip(ct, key))
print(pt.decode())
```

## Output

```text
DawgCTF{please_use_secrets_in_your_stream_ciphers_69bfe194af43f0cd}
```

## Flag

```text
DawgCTF{please_use_secrets_in_your_stream_ciphers_69bfe194af43f0cd}
```

## Why this challenge is weak

The main issue is that the keystream generator is **fully deterministic** after the initial random byte.

Once we recover even a small part of the keystream from known plaintext, we can predict the rest. Here, it is even worse because the generator quickly falls into a fixed point at `0xda`, so almost the entire keystream is the same repeated byte.

A secure stream cipher should generate keystream bytes that are unpredictable even if part of the plaintext is known.

## Takeaway

This challenge demonstrates why custom stream ciphers are dangerous:

- XOR encryption is only as strong as the keystream.
- Known plaintext instantly reveals keystream bytes.
- Weak state transitions can collapse into predictable or constant output.

The title **Six Seven** is a hint toward the repeated use of `6` and `7` inside the `gen()` function. # haha six seven very funny


* [🔙 Back to Crypto Directory](../Cryptography)
* [🔙 Back to Crypto Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../README.md)
