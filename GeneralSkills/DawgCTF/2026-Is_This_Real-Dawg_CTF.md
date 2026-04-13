## Challenge 5 — Is This Real?

### Idea

This challenge switches to asymmetric encryption. Alice asks Bob to encrypt the flag under Alice’s public key, but Bob does not verify that the supplied public key really belongs to Alice. That lets us substitute our own public key while still claiming to be Alice, so Bob encrypts the flag to us.


### Solve script

```python
#!/usr/bin/env python3
import requests

BASE = "https://protocols.live"

def new_instance(chal_no: int) -> int:
    r = requests.post(f"{BASE}/model/{chal_no}", timeout=15)
    r.raise_for_status()
    return r.json()["conn_id"]

def send(endpoint: str, conn_id: int, content: str) -> str:
    r = requests.post(
        f"{BASE}/{endpoint}",
        json={"conn_id": conn_id, "content": content},
        headers={"Content-Type": "application/json"},
        timeout=15,
    )
    r.raise_for_status()
    return r.json()["content"]

def util(path: str, content: str) -> str:
    r = requests.post(
        f"{BASE}/util/{path}",
        json={"conn_id": 0, "content": content},
        headers={"Content-Type": "application/json"},
        timeout=15,
    )
    r.raise_for_status()
    return r.json()["content"]

def parse_items(content: str):
    out = []
    for item in content.split("|"):
        typ, val = item.split(":", 1)
        out.append((typ, val))
    return out

def main():
    cid = new_instance(5)

    kp = util("gen_asym_key_pair", "")
    items = parse_items(kp)
    pub = items[1][1]
    priv = items[3][1]

    alice_msg = send("alice", cid, "t:x")
    print(f"[+] Alice says: {alice_msg}")

    forged_parts = parse_items(alice_msg)
    out = []
    replaced = False
    for typ, val in forged_parts:
        if typ == "k" and not replaced:
            out.append(f"k:{pub}")
            replaced = True
        else:
            out.append(f"{typ}:{val}")
    forged = "|".join(out)

    print(f"[+] Forged for Bob: {forged}")

    bob_msg = send("bob", cid, forged)
    print(f"[+] Bob says: {bob_msg}")

    ciphertext = None
    for typ, val in parse_items(bob_msg):
        if typ == "d":
            ciphertext = val
            break

    dec = util("asym_decrypt", f"k:{priv}|d:{ciphertext}")
    print(f"[+] Decrypted: {dec}")

if __name__ == "__main__":
    main()
```

---## Conclusion

Challenges 1 through 5 all fail because the protocol scripts trust message contents more than they trust identities.

- In Challenges 1–3, plaintext impersonation is enough.
- In Challenge 4, symmetric encryption is useless because the key and nonce are exposed.
- In Challenge 5, asymmetric encryption is misused because Bob accepts any supplied public key without verifying ownership.

The common lesson is that cryptography alone does not make a protocol secure. The protocol must also authenticate who is speaking and bind identities to the right keys.


* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
