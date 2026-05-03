# Mobile Waf Writeup

## Challenge Info

- **Name:** Mobile Waf
- **Category:** Misc
- **Points:** 990
- **Flag format:** `KubSTU(...)`

## Summary

The service presents 100 HTTP requests over a raw TCP connection and asks us to classify each one as either `Block` or `Allow` without making a single mistake.

At first glance this looks like a standard WAF-signature task, but that approach is not reliable enough. Some requests contain strings that look malicious while still being labeled safe, depending on endpoint and context. The clean solution is to turn the service into a labeling oracle, collect enough samples, and then automate the full run with a request-template classifier.

The final flag was:

```text
KubSTU(y0u_4r3_4_g00d_m0b1l3_4551574n7_f0r_d373c71ng_3v1l)
```

## Initial Recon

Connecting to the service shows:

```text
=== WAF Challenge ===
You need to correctly classify 100 HTTP requests as malicious or safe.
For each request, respond with:
  - 'Block' if the request is malicious
  - 'Allow' if the request is safe
Type 'Start' to begin:
```

After sending `Start`, the service prints one full HTTP request and waits for input:

```text
--- Request 1/100 ---
POST /api/search HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 89

{"query":"test' UNION SELECT NULL,NULL,NULL--","filter":"active"}
Your answer (Block/Allow):
```

If the answer is wrong, the server immediately reveals the true class:

```text
✗ Wrong! The request was MALICIOUS.
Correct answers so far: 0/100
Challenge failed. Try again!
```

That message is the key. It means every fresh connection can be used to obtain one correctly labeled sample.

## Key Observations

### 1. The server leaks labels on failure

By intentionally answering only the first request on each new connection, I could harvest labeled examples very quickly:

- Send `Start`
- Read request `1/100`
- Answer with either `Allow` or `Block`
- If wrong, record the leaked label from `The request was SAFE/MALICIOUS`
- If correct, infer the label from the chosen answer

This turns the challenge into a black-box data collection problem.

### 2. Simple keyword rules are not enough

Several requests that look suspicious are actually marked safe. For example:

- `GET /api/search?q=union+select+null`
- `GET /api/test?id=1' OR '1'='1`
- `GET /api/exec?cmd=ls`
- `POST /api/transform` with a normal JSON `xpath`

At the same time, genuinely malicious variants exist for SQLi, XSS, path traversal, XXE, template injection, unsafe code execution, and malicious upload bodies.

So a naive rule like "if it contains `UNION` then block" fails.

### 3. The request pool is finite and repetitive

I sampled the service repeatedly across the three provided hosts and built two datasets:

- `mobile_waf_dataset600.json`
- `mobile_waf_dataset1200_more.json`

Combined results:

- `1800` labeled samples
- `158` distinct exact requests
- `95` canonical request templates
- `0` label conflicts

The absence of conflicts strongly suggests the challenge draws from a fixed request set.

## Exploitation Strategy

The final solver used three layers:

### 1. Exact request lookup

If a request exactly matched one previously labeled sample, the solver reused that label immediately.

### 2. Canonical template lookup

Many requests only differed in non-essential fields such as `Host`, `Authorization`, `User-Agent`, or `X-Forwarded-For`.

So each request was canonicalized to:

- request line
- `Content-Type` header when present
- body

This preserved the fields that actually changed the semantics of the request while ignoring noisy headers.

### 3. Fallback heuristics

For anything unseen, the solver applied conservative rules:

- Mark obvious traversal, XXE, XSS, SQLi, command execution, or webshell uploads as malicious
- Mark known benign query examples and harmless API calls as safe

In practice, the exact and canonical maps handled almost everything.

## Why Canonicalization Worked

A typical safe request and malicious request often differed only in the path/body logic, not the surrounding headers.

For example, these were stable after normalization:

- Safe uploads with `filename="image.jpg"`
- Malicious uploads with `filename="shell.php"`
- Safe `POST /api/eval` with `{"expression":"2+2","type":"math"}`
- Malicious `POST /api/eval` that read `/etc/passwd`

Once the body was retained and irrelevant headers were dropped, the templates remained uniquely labeled.

## Solver Outline

The full script is saved at:

- [mobile_waf_solver.py](/home/priyanshu/mobile_waf_solver.py)

Core logic:

```python
def classify(req, exact_map, canon_map):
    req = normalize_newlines(req)
    if req in exact_map:
        return exact_map[req], "exact"

    creq = canonicalize_request(req)
    if creq in canon_map:
        return canon_map[creq], "canonical"

    return heuristic_label(req), "heuristic"


def canonicalize_request(req):
    req = req.replace("\r\n", "\n").strip("\n")
    lines = req.split("\n")

    if "" in lines:
        split_at = lines.index("")
        headers = lines[1:split_at]
        body = "\n".join(lines[split_at + 1:])
    else:
        headers = lines[1:]
        body = ""

    keep_headers = []
    for header in headers:
        if ":" not in header:
            continue
        name, value = header.split(":", 1)
        if name.lower() in {"content-type"}:
            keep_headers.append(f"{name}:{value.strip()}")

    parts = [lines[0], *sorted(keep_headers)]
    if body:
        parts.extend(["", body])
    return "\n".join(parts)
```

The interactive loop was straightforward:

1. Connect to one of the challenge hosts
2. Send `Start`
3. Read each HTTP request until `Your answer (Block/Allow):`
4. Classify via exact lookup, canonical lookup, then heuristic fallback
5. Send `Block` for malicious or `Allow` for safe
6. Repeat until the flag appears

The solver also supports learning from failure: if an unseen request causes a miss, the server reveals the true class, and the script stores that template for the next attempt.

## Final Run

The final automated run succeeded cleanly and produced:

```text
KubSTU(y0u_4r3_4_g00d_m0b1l3_4551574n7_f0r_d373c71ng_3v1l)
```

## Takeaway

This challenge was less about building a real WAF and more about recognizing the interaction model:

- the server reveals the ground truth on mistakes
- the request distribution is finite
- suspicious substrings alone are not enough

Once those properties are exploited, the problem becomes a repeatable data-labeling and template-matching task rather than a fragile hand-written signature exercise.



---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
