# DawgCTF Protocol Analysis 8 - Reflection

## Challenge
Bob and Alice both operate in effectively mirrored protocol roles, which creates a reflection/oracle style flaw in how signed nonces are accepted.

Flag format: `DawgCTF{}`

## Vulnerability
Alice signs attacker-controlled identity/nonce context in step 2:

- Alice receives: `pubX, X, certX, nX1`
- Alice responds with: `nA, SIGN_A(X, nX1, nA)`

Bob later accepts a signed tuple in his final verification path. By carefully reflecting Bob's own identity material into Alice's signing step, we can get Alice to produce a signature Bob accepts, without knowing Alice's private key.

## Exploit Flow (Working)
Use a single `conn_id` for challenge 8.

1. `POST /model/8` -> `conn_id = C`
2. `POST /bob` with empty content -> `B1 = pubB|n:bob|certB`
3. `POST /alice` with empty content -> `A1 = pubA|n:alice|certA`
4. Send Bob: `A1|d:<any 32-byte nonce>`  
   Bob returns: `nB|SIGN_B(...)`
5. Send Alice: `B1|d:nB`  
   Alice returns: `nA2|SIGN_A(bob, nB, nA2)` (effective tuple she signs)
6. Forward Alice's response from step 5 to Bob
7. Bob returns the flag

## Solver (Python)
```python
import requests

BASE = "https://protocols.live"

def post(path, conn_id, content=""):
    r = requests.post(
        f"{BASE}/{path}",
        json={"conn_id": conn_id, "content": content},
        headers={"Content-Type": "application/json"},
        timeout=10,
    )
    r.raise_for_status()
    return r.json()["content"]

# 1) new instance
conn_id = requests.post(f"{BASE}/model/8", json={}).json()["conn_id"]

# 2) first sends
b1 = post("bob", conn_id, "")     # pubB|bob|certB
a1 = post("alice", conn_id, "")   # pubA|alice|certA

# 3) advance Bob and get nB
any_nonce = "00" * 32
b2 = post("bob", conn_id, f"{a1}|d:{any_nonce}")
nB = b2.split("|")[0].split(":", 1)[1]

# 4) ask Alice to sign reflected Bob tuple
a2 = post("alice", conn_id, f"{b1}|d:{nB}")

# 5) finalize with Bob
flag_msg = post("bob", conn_id, a2)
print(flag_msg)
```

## Flag
`DawgCTF{4SK_4ND_U_SH4LL_R3C31V3}`

---
* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
