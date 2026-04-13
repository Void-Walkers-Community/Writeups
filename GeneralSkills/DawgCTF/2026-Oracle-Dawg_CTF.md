protocol analysis 9 : oracle :- DawgCTF{ST4R3_1NTO_TH3_VO1D}

# DawgCTF Protocol Analysis 9: Oracle

## Flag
`DawgCTF{ST4R3_1NTO_TH3_VO1D}`

## Idea
Bob sends Alice:
- `{{FLAG}pubA, B}pubA, A`

Alice then enters an infinite loop where she accepts:
- `pubX, X, certX, {{m}pubA, X}pubA, A`

and responds:
- `pubA, A, certA, {{m}pubX, A}pubX, X`

This is a decryption oracle:
1. We choose attacker keypair `(pubX, privX)` and a valid cert for name `mallory`.
2. We wrap any ciphertext `C` (encrypted to Alice) as `Enc_pubA("d:C|n:mallory")`.
3. Alice decrypts with `privA`, re-encrypts the recovered `m` to `pubX`, and returns it.
4. We decrypt Alice’s response with `privX` to recover `m`.

Bob’s payload has two nested encryptions under Alice’s key, so run oracle-decrypt twice:
1. Decrypt outer `C` -> get `d:<inner>|n:bob`
2. Decrypt `<inner>` -> get `t:DawgCTF{...}`

## Solve Script (Python)
```python
import requests

BASE = "https://protocols.live"
UTIL = BASE + "/util"

def post(url, conn_id, content):
    r = requests.post(url, json={"conn_id": conn_id, "content": content}, timeout=20)
    r.raise_for_status()
    return r.json()["content"]

def parts(content):
    return [tuple(x.split(":", 1)) for x in content.split("|")]

def vals(content, typ):
    return [v for t, v in parts(content) if t == typ]

def asym_enc(pub, text):
    return vals(post(UTIL + "/asym_encrypt", 0, f"k:{pub}|t:{text}"), "d")[0]

def asym_dec(priv, data_hex):
    return post(UTIL + "/asym_decrypt", 0, f"k:{priv}|d:{data_hex}")

conn = requests.post(BASE + "/model/9", json={}, timeout=20).json()["conn_id"]

# normal start: get Alice hello, then Bob message containing nested ciphertext
m1 = post(BASE + "/alice", conn, "")
m2 = post(BASE + "/bob", conn, m1)
pubA = vals(m1, "k")[0]
outer_from_bob = vals(m2, "d")[1]

# attacker identity
kp = post(UTIL + "/gen_asym_key_pair", 0, "")
pubX, privX = vals(kp, "k")
name = "mallory"
certX = vals(post(UTIL + "/get_cert", 0, f"k:{pubX}|n:{name}"), "d")[0]

def oracle_dec(cipher_under_pubA):
    wrapped = asym_enc(pubA, f"d:{cipher_under_pubA}|n:{name}")
    req = f"k:{pubX}|n:{name}|d:{certX}|d:{wrapped}|n:alice"
    resp = post(BASE + "/alice", conn, req)
    resp_ct = parts(resp)[3][1]
    p1 = asym_dec(privX, resp_ct)      # d:<inner>|n:alice
    inner = parts(p1)[0][1]
    p2 = asym_dec(privX, inner)        # recovered m
    return p2

step1 = oracle_dec(outer_from_bob)      # d:<inner_flag_ct>|n:bob
inner_flag_ct = parts(step1)[0][1]
flag_msg = oracle_dec(inner_flag_ct)    # t:DawgCTF{...}
print(flag_msg)
```

## Why It Works
Alice acts as a chosen-ciphertext re-encryption oracle: anything validly encrypted to Alice can be transformed into something decryptable by us, because Alice re-encrypts plaintext to our supplied public key.


* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
