```
#!/usr/bin/env python3
import random
import socket
import sys
import time

HOST = "wandering.ctf.theromanxpl0.it"
PORT = 9091
N = 256

BASE = list(range(1, N, 2)) + list(range(0, N, 2))


def nqueens_with_hidden(a, b):
    """Return row -> column solution containing the two hidden queens."""
    fixed = {127 + a: 129 + b, 23 + a: 45 + b}
    rng = random.Random((a + 8) * 100 + b + 8)
    q = BASE[:]

    for row, col in fixed.items():
        other = q.index(col)
        q[row], q[other] = q[other], q[row]

    fixed_rows = set(fixed)
    nonfixed = [r for r in range(N) if r not in fixed_rows]

    def make_counts():
        d1 = [0] * (2 * N - 1)
        d2 = [0] * (2 * N - 1)
        for r, c in enumerate(q):
            d1[r + c] += 1
            d2[r - c + N - 1] += 1
        return d1, d2

    d1, d2 = make_counts()

    def row_conflicts(r):
        c = q[r]
        return (d1[r + c] - 1) + (d2[r - c + N - 1] - 1)

    def attacks():
        return sum(c * (c - 1) // 2 for c in d1) + sum(c * (c - 1) // 2 for c in d2)

    def swap_delta(r, s):
        if r == s:
            return 10**9
        cr, cs = q[r], q[s]
        touched = {}
        for typ, k in (
            ("a", r + cr),
            ("b", r - cr + N - 1),
            ("a", s + cs),
            ("b", s - cs + N - 1),
            ("a", r + cs),
            ("b", r - cs + N - 1),
            ("a", s + cr),
            ("b", s - cr + N - 1),
        ):
            if (typ, k) not in touched:
                touched[(typ, k)] = d1[k] if typ == "a" else d2[k]

        after = touched.copy()
        after[("a", r + cr)] -= 1
        after[("b", r - cr + N - 1)] -= 1
        after[("a", s + cs)] -= 1
        after[("b", s - cs + N - 1)] -= 1
        after[("a", r + cs)] = after.get(("a", r + cs), d1[r + cs]) + 1
        after[("b", r - cs + N - 1)] = after.get(("b", r - cs + N - 1), d2[r - cs + N - 1]) + 1
        after[("a", s + cr)] = after.get(("a", s + cr), d1[s + cr]) + 1
        after[("b", s - cr + N - 1)] = after.get(("b", s - cr + N - 1), d2[s - cr + N - 1]) + 1

        before_pairs = sum(c * (c - 1) // 2 for c in touched.values())
        after_pairs = sum(c * (c - 1) // 2 for c in after.values())
        return after_pairs - before_pairs

    total = attacks()
    for _ in range(100000):
        if total == 0:
            return q

        conflicted = [r for r in nonfixed if row_conflicts(r) > 0]
        r = rng.choice(conflicted or nonfixed)
        best_delta = 10**9
        best_rows = []
        for s in nonfixed:
            delta = swap_delta(r, s)
            if delta < best_delta:
                best_delta = delta
                best_rows = [s]
            elif delta == best_delta:
                best_rows.append(s)

        s = rng.choice(best_rows)
        cr, cs = q[r], q[s]
        for rr, cc, sign in ((r, cr, -1), (s, cs, -1)):
            d1[rr + cc] += sign
            d2[rr - cc + N - 1] += sign
        q[r], q[s] = q[s], q[r]
        for rr, cc, sign in ((r, cs, 1), (s, cr, 1)):
            d1[rr + cc] += sign
            d2[rr - cc + N - 1] += sign
        total += best_delta

    raise RuntimeError(f"failed to solve for offset {(a, b)}")


def payload_for(a, b):
    q = nqueens_with_hidden(a, b)
    hidden_rows = {127 + a, 23 + a}
    return "".join(f"{q[row]},{row}\n" for row in range(N) if row not in hidden_rows).encode()


def main():
    payloads = {(a, b): payload_for(a, b) for a in range(-4, 4) for b in range(-4, 4)}
    order = list(payloads)
    random.shuffle(order)

    attempt = 0
    with socket.create_connection((HOST, PORT), timeout=10) as s:
        s.settimeout(2)
        while True:
            attempt += 1
            guess = random.choice(order)
            print(f"attempt {attempt}: hidden offset guess {guess}", flush=True)
            s.sendall(payloads[guess])
            out = b""
            deadline = time.time() + 3
            while time.time() < deadline:
                try:
                    chunk = s.recv(4096)
                except socket.timeout:
                    continue
                if not chunk:
                    print(out.decode(errors="replace"))
                    return 1
                out += chunk
                text = out.decode(errors="replace")
                if "TRX{" in text or "incorrect!" in text or "Failed initialization" in text:
                    print(text, end="" if text.endswith("\n") else "\n", flush=True)
                    if "TRX{" in text:
                        return 0
                    break
            else:
                print("no response; reconnect and retry", file=sys.stderr)
                return 2


if __name__ == "__main__":
    raise SystemExit(main())
```
Flag = TRX{1_v3_b33n_w4nd3r1ng_4ll_th1s_ch3ssb0ard_but_no_b4cktr4ck1ng}

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

