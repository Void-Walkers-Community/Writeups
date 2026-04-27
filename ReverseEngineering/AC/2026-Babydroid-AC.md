# babydroid

**Category:** Misc  
**Points:** 100  
**Author:** stancium  
**Solved on:** April 26, 2026  
**Active instance used:** `http://1343777b8ead1cef.ctf.ac.upt.ro`

## Summary

The APK contains two important network paths:

1. A hidden startup request to `/v1/hidden/flag`
2. A user-facing "key fetch" request to `/takei/ppv1?n=7&shift=3`

The first path returns the flag encrypted with AES-GCM.  
The second path looks broken at first because it returns HTTP 404, but the app still reads the 404 HTML body and turns that body into key material. Once that behavior is reproduced exactly, the hidden blob decrypts cleanly to the flag.

## Static Analysis

After decompiling the APK with JADX, the relevant files were:

- `decompiled/sources/com/example/baby/GhostBoxKt.java`
- `decompiled/sources/defpackage/qa.java`
- `decompiled_bad/sources/defpackage/mw.java`
- `decompiled/sources/defpackage/s00.java`
- `decompiled/sources/defpackage/qh0.java`
- `decompiled_bad/sources/defpackage/dl0.java`
- `decompiled_bad/sources/defpackage/di.java`

### 1. Hidden startup request

`GhostBoxKt.ShadowPane(...)` launches a coroutine on startup. In `qa.java`, case `2`, the code builds:

```java
String strO = dl0.o("BA0DHg==");      // flag
String strO2 = dl0.o("TRdTVg==");      // /v1/
String strO3 = dl0.o("CggGHTod");      // hidden
String str2 = md0.X(str, '/') + strO2 + strO3 + "/" + strO;
```

So the hidden route is:

```text
/v1/hidden/flag
```

Requesting that route returns a Base64 blob, not plaintext.

### 2. Encryption routine

In `qh0.b(String plaintext, String keyMaterial)`, the app encrypts using:

- `AES/GCM/NoPadding`
- 12-byte random IV
- AES key = `SHA-256(keyMaterial)[:16]`
- output = `Base64(iv || ciphertext || tag)`

That told us the `/v1/hidden/flag` response was almost certainly an encrypted flag.

### 3. Key-fetch route

In `mw.b(...)`, the app constructs the visible key endpoint:

```text
/takei/ppv1?n=7&shift=3
```

After fetching the response, the app derives the final key material as:

```text
reverse(di.v(response_body))
```

where `di.v(...)` is a Caesar shift of `+3` on alphabetic characters.

## Important Implementation Detail

The solve hinges on `s00.java`.

The request code does **not** fail on non-2xx responses. It calls `getErrorStream()` for error codes and still reads the body:

```java
BufferedReader bufferedReader = new BufferedReader(
    new InputStreamReader(
        (200 > responseCode || responseCode >= 300)
            ? httpURLConnection.getErrorStream()
            : httpURLConnection.getInputStream()
    )
);
```

It also reads the body line-by-line with `readLine()` and appends each line directly, which means newline characters are removed.

That means the 404 HTML from `/takei/ppv1?n=7&shift=3` is not an error condition for the app. It is effectively the input used to derive the AES key.

## Exploitation

The solve steps are:

1. Request `/takei/ppv1?n=7&shift=3`
2. Take the 404 HTML body
3. Remove line breaks exactly like the app does
4. Apply `di.v(...)`
5. Reverse the result
6. Hash with SHA-256 and keep the first 16 bytes
7. Request `/v1/hidden/flag`
8. Base64-decode and decrypt with AES-GCM

## Solver

```python
import base64
import hashlib
import requests
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

BASE = "http://1343777b8ead1cef.ctf.ac.upt.ro"

def di_v(s: str) -> str:
    out = []
    for ch in s:
        o = ord(ch)
        if 97 <= o < 123:
            out.append(chr(((o - 94) % 26) + 97))
        elif 65 <= o < 91:
            out.append(chr(((o - 62) % 26) + 65))
        else:
            out.append(ch)
    return "".join(out)

key_resp = requests.get(f"{BASE}/takei/ppv1?n=7&shift=3", timeout=5)
key_body = "".join(key_resp.text.splitlines())
key_material = di_v(key_body)[::-1]

blob = requests.get(f"{BASE}/v1/hidden/flag", timeout=5).text.strip()
raw = base64.b64decode(blob)
iv, ct = raw[:12], raw[12:]

key = hashlib.sha256(key_material.encode()).digest()[:16]
flag = AESGCM(key).decrypt(iv, ct, None).decode()
print(flag)
```

## Flag

```text
CTFAC{95695866e66f9cfba7030c25d1a95e69b05443cf8e3d0884cfc60a4eddff9cc1}
```

## Notes

The main trap is assuming the key endpoint must return HTTP 200. In reality, the app intentionally accepts the 404 page and converts that HTML into the AES key source. Missing the newline-stripping behavior also causes decryption to fail.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
