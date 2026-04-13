## Challenge 2 — Liar

### Idea

This challenge is almost the same as Challenge 1, except Bob expects the sender name to be `charlie` instead of `alice`. Alice still generates a normal request claiming to be Alice, so we intercept it and change only the sender name before forwarding it.


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

def main():
    cid = new_instance(2)

    alice_msg = send("alice", cid, "t:x")
    print(f"[+] Alice says: {alice_msg}")

    forged = alice_msg.replace("n:alice", "n:charlie")
    print(f"[+] Forged for Bob: {forged}")

    bob_msg = send("bob", cid, forged)
    print(f"[+] Bob says: {bob_msg}")

if __name__ == "__main__":
    main()
```

## Conclusion

Challenges 1 through 5 all fail because the protocol scripts trust message contents more than they trust identities.

- In Challenges 1–3, plaintext impersonation is enough.
- In Challenge 4, symmetric encryption is useless because the key and nonce are exposed.
- In Challenge 5, asymmetric encryption is misused because Bob accepts any supplied public key without verifying ownership.

The common lesson is that cryptography alone does not make a protocol secure. The protocol must also authenticate who is speaking and bind identities to the right keys.


* [🔙 Back to GeneralSkills Directory](../GeneralSkills)
* [🔙 Back to General Skills Index Directory](../GeneralSkills/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
