## Challenge 4 — Real Security!

### Idea

Alice asks Bob to encrypt the flag using a symmetric key and nonce that she sends in plaintext. Bob then returns the ciphertext. Since the attacker can read the exact key and nonce that Alice provided, the ciphertext can be decrypted with the helper utility.

### Important live-service note

On the live service, Bob was picky about the exact wording of Alice’s message. Handcrafting the request caused `400 Invalid message` errors. The reliable method was to first ask Alice for her exact request string and then relay it unchanged to Bob.


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
    cid = new_instance(4)

    alice_msg = send("alice", cid, "t:x")
    print(f"[+] Alice says: {alice_msg}")

    key = None
    nonce = None
    for typ, val in parse_items(alice_msg):
        if typ == "k" and key is None:
            key = val
        elif typ == "d" and nonce is None:
            nonce = val

    bob_msg = send("bob", cid, alice_msg)
    print(f"[+] Bob says: {bob_msg}")

    ciphertext = None
    for typ, val in parse_items(bob_msg):
        if typ == "d":
            ciphertext = val
            break

    dec = util("sym_decrypt", f"k:{key}|d:{nonce}|d:{ciphertext}")
    print(f"[+] Decrypted: {dec}")

if __name__ == "__main__":
    main()
```

---

## Conclusion

Challenges 1 through 5 all fail because the protocol scripts trust message contents more than they trust identities.

- In Challenges 1–3, plaintext impersonation is enough.
- In Challenge 4, symmetric encryption is useless because the key and nonce are exposed.
- In Challenge 5, asymmetric encryption is misused because Bob accepts any supplied public key without verifying ownership.

The common lesson is that cryptography alone does not make a protocol secure. The protocol must also authenticate who is speaking and bind identities to the right keys.


* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
