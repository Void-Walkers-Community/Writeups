# Writeup

## Summary

The challenge exposes three endpoints:

- `GET /` creates a random `uid` with `0` studs.
- `GET /buy?uid=...` gives a free signature for `0|uid`, and returns the flag once the user has at least `7` studs.
- `POST /work` verifies a signature for the current `studs|uid` payload and increments the user's stud count.

The intended difficulty is the HAProxy rate limit:

```haproxy
stick-table type string len 2048 size 100k expire 20s store http_req_rate(20s)
http-request track-sc0 url
http-request deny deny_status 429 if { sc_http_req_rate(0) gt 1 }
```

This rate limits by the full URL, not by client IP or route. Adding a unique query string to every request bypasses it immediately.

## Bugs

There are two important bugs in `backend/app.py`.

### 1. Rate limit bypass

Because HAProxy tracks the full URL, these are treated as different requests:

- `/work?a=1`
- `/work?a=2`
- `/work?a=3`

So we can make as many requests as we want by changing the query string each time.

### 2. Signature cache keyed by hex string, not by signature bytes

In `/work`, the server does:

```python
sig = str(request_body["sig"])
sig_bytes = bytes.fromhex(sig)
value = r.get(str(sig))
```

The Redis cache key is the exact hex string, but verification uses the decoded bytes. Hex decoding is case-insensitive, so these all decode to the same bytes:

- `deadbeef`
- `DEADBEEF`
- `dEaDbEeF`

That means one valid signature can be replayed many times by changing only the casing of `a-f`.

## Important control flow

The critical part of `/work` is:

```python
if verified:
    studs = r.incr(uid)
    if studs > 2:
        return "You're not getting any more free studs!"
    else:
        new_sig = uov.sign(str(studs) + '|' + uid)
        r.set(new_sig, str(studs) + '|' + uid, ex=240)
        return f"Your next free stud is {new_sig}!"
```

This is backwards for security:

- verification is done against the current payload
- then `r.incr(uid)` happens unconditionally
- only after incrementing does the server check whether `studs > 2`

So a valid signature for `2|uid` is enough to keep incrementing the counter past `2`, even though the response says no more free studs.

## Exploit

### Step 1. Create a user

Call `/` once:

```text
Your randomly generated uid is <uid>!
```

### Step 2. Get the free `0|uid` signature

Call:

```text
/buy?uid=<uid>
```

This returns a valid signature for payload `0|uid`.

### Step 3. Walk to 2 studs normally

Use the `0|uid` signature once on `/work` to get a valid signature for `1|uid`.

Use the `1|uid` signature once on `/work` to get a valid signature for `2|uid`.

At this point we have a real valid signature for the current payload.

### Step 4. Replay that same signature many times with different hex casing

Because Redis caches by the literal hex string, we can submit many case-variants of the same `2|uid` signature:

- original lowercase
- uppercase one `a`
- uppercase two different `a-f` positions
- and so on

Each variant is:

- a cache miss in Redis
- the same signature bytes after `bytes.fromhex`
- valid for `2|uid`

The server increments `uid` for every accepted request, even after the count passes `2`.

So after enough case-variants, the user reaches `7` studs.

### Step 5. Buy the Lego set

Call:

```text
/buy?uid=<uid>
```

Now the service returns the flag.

## Why this works reliably

The exploit does **not** need to break UOV and does **not** need a fragile race.

The real issue is the mismatch between:

- cache key: exact hex string
- verifier input: decoded bytes

Once we have one valid signature for `2|uid`, we can replay it through many distinct Redis keys just by changing hex casing.

## Script

The solve script is:

- [solve_legos.py](/home/parth/HACK/solve_legos.py)

Usage:

```bash
python3 /home/parth/HACK/solve_legos.py http://107.178.211.79
python3 /home/parth/HACK/solve_legos.py http://hensandroosters.crypto.ctf.umasscybersec.org
```

## Flag

```text
UMASS{oil_does_mix_with_oil_but_roosters_dont}
```

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
