# FS-Enc — Reverse Engineering Writeup

## Challenge

Two files were served from `http://10.83.18.160:8080/`:

| File | Size | SHA-256 |
|---|---:|---|
| `fs-enc` | 5,349,400 bytes | `58296F1F0F27AA0A544BC79475EE4CD36E5E15130F0CAB117A11294B4FB8ECC2` |
| `flag.bin` | 114 bytes | `785DBE5AAE635734B78A707637E06AC90CC83F881188680EFE98F151F50759FD` |

Goal: decrypt `flag.bin` and recover the `PP{...}` flag.

## 1. Identifying the binary

`fs-enc` is a non-stripped x86-64 Linux ELF with Rust symbols:

```
ELF 64-bit LSB executable, x86-64
Rust crate: fs_enc
Symbols:    Vault::open, Vault::print, read_magic, ...
Libraries:  argon2, chacha20poly1305, aead, rpassword, clap, zeroize, anyhow
```

The crypto dependencies immediately suggest the scheme:

- **Argon2** for password-based key derivation
- **ChaCha20-Poly1305** for authenticated encryption

Relevant `.rodata` strings:

```
Creating encrypted file:
Printing content of encrypted file:
Modifying content of encrypted file:
This is your super secret vault
Key invalid
invalid password length (0)
```

The CLI exposes `create`, `print`, `padding`, `modify`, `set`, and options such as `--password` / `PASSWORD`.

## 2. Locating the magic value

`read_magic` at `0x2302e0` returns a 12-byte constant from address `0x217124` in `.debug_magic`:

```
e7 4c 97 2e 3a e8 ce d4 2b f0 42 7a
```

These are exactly the first 12 bytes of `flag.bin`, so the magic is a fixed binary constant, not a random salt:

```
$ xxd -l 16 flag.bin
00000000: e74c 972e 3ae8 ced4 2bf0 427a d16f  ace5  .L..:...+ .Bz.o..
```

## 3. Reconstructing the key derivation

The main path (around `0x2309a2`) does:

```c
magic = read_magic();                  // 12 bytes
salt[0..12] = magic;
salt[12..16] = magic[0..4];            // salt = magic || magic[0:4]

Argon2::default();
argon2.hash_password_into(
    password, password_len,
    salt, 16,
    out, 44                            // 44-byte derived material
);
```

Two observations:

1. The salt is deterministic and entirely contained in the file.
2. The KDF output is **44 bytes**, not the usual 32.

## 4. File format

`flag.bin` is exactly 114 bytes:

```
12-byte magic
44-byte Argon2-derived material
58-byte ChaCha20-Poly1305 output
```

The 44-byte material splits into:

```
[ 32-byte ChaCha20 key ][ 12-byte nonce ]
```

Complete layout:

| Offset | Size | Field |
|---|---:|---|
| `0x00` | 12 | Magic (`e74c972e3ae8ced42bf0427a`) |
| `0x0C` | 32 | ChaCha20 key |
| `0x2C` | 12 | ChaCha20-Poly1305 nonce |
| `0x38` | 42 | Ciphertext |
| `0x62` | 16 | Poly1305 tag |

This matches `114 = 12 + 32 + 12 + 42 + 16`, with a 42-byte plaintext including the trailing newline.

Actual values:

```
magic  e74c972e3ae8ced42bf0427a
key    d16face575734b9540c12af19b97a4210d7b15cda29f81410c9c84461f5d3d4b
nonce  03b9f7a012a72cffd575b2b3
sealed 37436fbd2db85e13a8a6122cb0559267685d03ca0c30910f7b2a20a95bae67da
       9f7187ab934798541cd31e5c1e60af402c717dfdecae64260d67
```

## 5. The vulnerability

`Vault::open` at `0x232540` does roughly:

```c
file = File::open(path, "a+");

read_exact(&mut magic, 12);
assert!(magic == read_magic());

read_exact(&mut header, 44);           // key || nonce

// SIMD comparison of the caller-supplied 32-byte key
// against header[0..32]
if supplied_key != header[0..32] {
    bail!("Key invalid");
}

read_to_end(&mut ciphertext);          // ciphertext || tag
plaintext = ChaCha20Poly1305::new(supplied_key)
                .decrypt(header[32..44], &ciphertext);
```

The crucial mistake: `header[0..32]` is not a verifier, hash, or wrapped key. It is **the actual ChaCha20 key**, stored unencrypted. Likewise, `header[32..44]` is the actual nonce.

The password check only verifies that the caller's Argon2 output matches bytes already present in the file. Anyone who reads the file can skip Argon2 and the password entirely:

```
file = magic || key || nonce || AEAD(key, nonce, plaintext)
```

The password-derived KDF output protects nothing.

## 6. Exploit

Any ChaCha20-Poly1305 implementation can decrypt directly from the header. Using Python and `pycryptodome` (`work/solve.py`):

```python
from Crypto.Cipher import ChaCha20_Poly1305

data = open("work/flag.bin", "rb").read()
assert len(data) == 114

magic  = data[0:12]    # fixed magic, unused for decryption
key    = data[12:44]   # stored ChaCha20 key
nonce  = data[44:56]   # stored nonce
sealed = data[56:]     # ciphertext || 16-byte tag

cipher = ChaCha20_Poly1305.new(key=key, nonce=nonce)
plaintext = cipher.decrypt_and_verify(sealed[:-16], sealed[-16:])
print(plaintext.decode(), end="")
```

Output:

```
PP{f1l3_3ncrypt10n_1s_h4rd::U1suRmBV9RJE}
```

The Poly1305 tag verifies, confirming the plaintext is authentic.

## 7. About the password

Recovering the password is unnecessary. The file leaks the Argon2 output (`key || nonce`) that the password would derive, so decryption never requires running Argon2.

For completeness, the password could still be brute-forced offline by hashing candidates with:

```
Argon2::default(), salt = magic || magic[0:4], output length = 44
```

and comparing the first 32 bytes with `flag.bin[12:44]`. Common passwords such as `password`, `admin`, `secret`, `vault`, and `fs-enc` do not match. The intended weakness, however, is the plaintext key storage — not a weak password.

## Flag

```
PP{f1l3_3ncrypt10n_1s_h4rd::U1suRmBV9RJE}
```

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
