## Challenge 1 — Can You Hear Me?

### Idea

This challenge is just a plaintext request followed by a plaintext response. Alice asks Bob for the flag, and Bob returns it. Since Bob only checks whether the received message matches the expected contents, we can simply relay Alice’s exact request to Bob.


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
    cid = new_instance(1)

    alice_msg = send("alice", cid, "t:x")
    print(f"[+] Alice says: {alice_msg}")

    bob_msg = send("bob", cid, alice_msg)
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
