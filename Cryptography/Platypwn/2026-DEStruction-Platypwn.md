FLAG0 = `PP{D3S_c4nn07_s4v3_7runc473d_GCM_fr0m_c0llisi0ns::-ZKa-Gw4ng2O}`
Raw bytes: `50507b4433535f63346e6e30375f733476335f3772756e63343733645f47434d5f6672306d5f63306c6c697369306e733a3a2d5a4b612d4777346e67324f7d`

How FLAG0 was obtained: the tag is only 16 bits (TAG_LENGTH = 2). Each connection: request one encryption with plaintext 00 12, recover the 12-byte keystream, XOR it with "give me flag" to get a valid command ciphertext under that nonce, then spend the remaining 167 attempts guessing tags. That is 167/65536 success per connection; it hit on connection 65.

Script used: 
```python
import socket, time, statistics, random, sys

HOST, PORT = '10.83.19.212', 2004

def connect():
    s = socket.create_connection((HOST, PORT), timeout=10)
    s.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
    s.settimeout(10)
    return s

def recv_until(s, marker, buf=b''):
    while marker not in buf:
        d = s.recv(65536)
        if not d:
            break
        buf += d
    return buf

def parse_enc(out):
    for line in out.decode(errors='replace').splitlines():
        if 'Here is a result' in line:
            return tuple(line.rsplit(': ', 1)[1].split(','))
    return None

def timed_submit(s, payload, buf):
    t0 = time.perf_counter()
    s.sendall(b'2\n' + payload.encode() + b'\n')
    buf = recv_until(s, b'Attempts left', buf)
    t1 = time.perf_counter()
    # drain the rest of the prompt up to choice:
    buf = recv_until(s, b'choice: ', buf)
    return t1 - t0, buf

def get_enc(s):
    buf = recv_until(s, b'choice: ')
    s.sendall(b'1\n0\n' + b'00' * 12 + b'\n')
    out = recv_until(s, b'choice: ')
    p = parse_enc(out)
    if not p:
        raise RuntimeError('enc parse fail ' + repr(out[-200:]))
    return p, b''

def make_ref():
    s = connect()
    try:
        p, _ = get_enc(s)
        return p
    finally:
        s.close()

ref = make_ref()
nonce, ct, ad, tag = ref
print('reference tuple', ref, flush=True)

same_valid = []
same_invalid = []
cross_valid = []
cross_invalid = []

random.seed(1234)
N = 18
for ci in range(N):
    try:
        s = connect()
    except Exception as e:
        print('connect fail', e, flush=True)
        continue

    # encryption on this connection (attempt 1)
    (n2, c2, a2, t2), buf = get_enc(s)
    own_valid = f"{n2},{c2},{a2},{t2}"
    own_invalid = f"{n2},{c2},{a2},{random.randrange(65536):04x}"
    cross_valid_p = f"{nonce},{ct},{ad},{tag}"
    cross_invalid_p = f"{nonce},{ct},{ad},{random.randrange(65536):04x}"

    order = []
    for i in range(6):
        order.append(('sv', own_valid))
        order.append(('si', own_invalid))
        order.append(('cv', cross_valid_p))
        order.append(('ci', cross_invalid_p))
    random.shuffle(order)

    for kind, pl in order:
        try:
            dt, buf = timed_submit(s, pl, buf)
        except Exception as e:
            break
        if kind == 'sv': same_valid.append(dt)
        elif kind == 'si': same_invalid.append(dt)
        elif kind == 'cv': cross_valid.append(dt)
        else: cross_invalid.append(dt)
    s.close()
    if ci % 3 == 0:
        def m(x): return round(statistics.median(x)*1e6,1) if x else None
        print('conn', ci, 'medians us sv/si/cv/ci:', m(same_valid), m(same_invalid), m(cross_valid), m(cross_invalid), flush=True)

def report(name, a, b):
    if not a or not b:
        print(name, 'no data'); return
    ma, mb = statistics.median(a), statistics.median(b)
    print(f'{name}: median(A)={ma*1e6:.1f}us median(B)={mb*1e6:.1f}us diff={(ma-mb)*1e6:.1f}us n={len(a)},{len(b)}')
    # also mean
    print(f'   mean(A)={statistics.mean(a)*1e6:.1f} mean(B)={statistics.mean(b)*1e6:.1f} diff={(statistics.mean(a)-statistics.mean(b))*1e6:.1f}')

print('=== POSITIVE CONTROL (same connection, valid vs invalid tag) ===')
report('same', same_valid, same_invalid)
print('=== CROSS CONNECTION (reference tuple valid vs invalid tag) ===')
report('cross', cross_valid, cross_invalid)
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
