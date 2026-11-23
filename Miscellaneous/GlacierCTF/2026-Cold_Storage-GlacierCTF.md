# Cold Storage — GlacierCTF 2026

**Category:** Web / Cryptography / Date-Time  
**Difficulty:** 50 points  
**Author:** ansa  
**Status:** Exploit developed; final flag not confirmed

## Challenge Description

> Ice VII is pretty cool.

The challenge provided an encrypted archive of scientific samples using AES-128-GCM.

The objective was to open a protected record with:

```text
classification = restricted
sample = Ice-VII
```

Successfully authenticating this record would cause the server to return the flag.

## Initial Analysis

The challenge used a TCP-based instance launcher. After connecting to port `13379` and pressing Enter, the server generated a temporary HTTPS instance with a 600-second lifetime.

The source exposed these important endpoints:

- `/archive` — retrieve 32 encrypted records.
- `/api/import` — import one chosen sample.
- `/api/open` — decrypt and authenticate an encrypted record.

Each archived record contained its original timestamp, classification, sample, ciphertext, and authentication tag.

The application generated a new AES-128 key for each instance.

## Vulnerability: AES-GCM Nonce Reuse

The main vulnerability was found in `record-crypto.js`.

```javascript
function createNonce(recorded) {
  return createHash('sha256')
    .update(
      Temporal.Instant.from(recorded)
        .epochNanoseconds.toString()
    )
    .digest()
    .subarray(0, 12);
}
```

The server derived the GCM nonce from the normalized timestamp.

JavaScript Temporal treats these two timestamps as the same instant:

```text
1972-12-31T23:59:59Z
1972-12-31T23:59:60Z
```

However, the import endpoint checked duplicate records using their original timestamp strings.

Consequently, two records with different timestamp strings could receive the exact same AES-GCM nonce.

This violates the fundamental nonce-uniqueness requirement of AES-GCM.

## Exploitation Strategy

We retrieved the archive and searched for a timestamp ending in `:59Z`.

One instance contained:

```text
Original:
1972-12-31T23:59:59Z

Collision:
1972-12-31T23:59:60Z
```

We then imported a new record using the second timestamp and a slightly modified sample.

Because both timestamps normalize to the same instant, the server derives the same GCM nonce for both encryptions.

The original plaintext and the imported plaintext are known, allowing the attacker to recover the encryption keystream.

## Recovering the Authentication Key

AES-GCM uses the GHASH authentication function over GF(2^128).

For two 32-byte ciphertexts under the same nonce, with identical first blocks, the difference in their authentication tags satisfies:

```text
T1 XOR T2 = (C1_block2 XOR C2_block2) * H²
```

Here, H is the GCM authentication subkey.

Therefore:
... (73 lines left)

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
