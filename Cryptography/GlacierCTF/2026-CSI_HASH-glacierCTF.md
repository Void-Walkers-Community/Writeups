## CSI-HASH write-up

The service gives a 32-character challenge and asks for a different message with the same digest.

`CSI_HASH.digest()` splits messages into 32-byte blocks and applies `process_block` to each one. In `process_block`, the byte values control how many isogenies are applied. If the block contains only zero bytes, then `any(es)` is false, so the isogeny loop does nothing. The curve is returned unchanged: an all-zero block acts as the identity.

Let `m` be the challenge message. Since it is exactly one block:

```text
digest(m) = process_block(0, m)
```

Append one full zero block to make a different message:

```text
m' = m || 00 00 ... 00   (32 zero bytes)
```

The second block leaves the intermediate curve unchanged, so:

```text
digest(m') = process_block(process_block(0, m), 00...00)
           = process_block(0, m)
           = digest(m)
```

The comparison rejects only when the submitted message equals the challenge. The forged message is longer, so it passes that check.

### Exploit

Send the challenge bytes followed by exactly 32 raw NUL bytes and a newline. Use a socket because terminal input is not suitable for sending NUL bytes.

```python
import socket

HOST, PORT = "challs.glacierctf.com", 13378

with socket.create_connection((HOST, PORT), timeout=30) as sock:
    sock.settimeout(300)

    banner = b""
    while b"> " not in banner:
        banner += sock.recv(4096)

    print(banner.decode(errors="replace"), end="")

    challenge = next(
        line.split(b":", 1)[1].strip()
        for line in banner.splitlines()
        if b"Here is your challenge:" in line
    )

    sock.sendall(challenge + b"\x00" * 32 + b"\n")

    while True:
        data = sock.recv(4096)
        if not data:
            break
        print(data.decode(errors="replace"), end="")
```

The challenge source’s prime list has 32 entries, so the zero block must be 32 bytes. The server returned:

```text
gctf{ac0dbddb0eea3de5c7a04695487358a8c3034abf1758c4c9be95233476351cb0}
```
 
 ---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
