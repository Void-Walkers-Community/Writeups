# Faultline Writeup

## Challenge Information

- **Challenge:** Faultline
- **Category:** Reverse Engineering

## Overview

The challenge provides a Linux ELF binary named `faultline`. Running the binary without arguments shows that it is a command-line utility for evaluating a 12-character profile string.

```bash
$ ./faultline
FAULTLINE / seam optimizer

usage:
  ./faultline notes
  ./faultline score  <PROFILE>
  ./faultline trace  <PROFILE>
  ./faultline token  <PROFILE>
  ./faultline compare <PROFILE_A> <PROFILE_B>
  ./faultline nudge <PROFILE> <INDEX> <DELTA>
  ./faultline submit <PROFILE> <TOKEN>

alphabet: BCDFGHJKLMNPQRST
profile length: 12
historical lock score: 2026
```

From this output, the important constraints are:

- A valid profile is exactly **12 characters** long.
- Every character must come from the alphabet:

```text
BCDFGHJKLMNPQRST
```

- The target score is:

```text
2026
```

The goal is to find a valid profile with score `2026`, generate its token, and submit both to recover the flag.

## Binary Triage

Basic file inspection shows that the binary is a 64-bit statically linked ELF.

```bash
$ file faultline
faultline: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=3e8f13e9a62787f9c41d40970ddfc7cdd80182e8, for GNU/Linux 4.4.0, not stripped
```

Because the binary is **not stripped**, function names are visible. Decompiling `main()` shows several useful commands:

```text
notes
score
trace
token
compare
nudge
submit
```

The interesting functions are:

```text
parseProfile
computeFaultlineScoreVisible
buildSurveyTokenVisible
validate
```

The `score` command parses a profile and prints its score. The `token` command builds a token for a valid profile. The `submit` command checks the profile and token pair.

This suggests the intended solve path:

1. Find a valid profile with score `2026`.
2. Generate the token for that profile.
3. Submit the profile and token.

## Understanding the Profile Space

The alphabet has 16 characters:

```text
BCDFGHJKLMNPQRST
```

The profile length is 12, so the total search space is:

```text
16^12 = 281,474,976,710,656
```

A full brute force is not realistic.

However, the binary gives us an oracle:

```bash
./faultline score <PROFILE>
```

This means we can repeatedly query the score of candidate profiles and search for one that reaches the target score `2026`.

## Solver Strategy

Instead of brute forcing every possible profile, I used a hill-climbing search:

1. Start from a random valid 12-character profile.
2. Compute its score using `./faultline score`.
3. Try changing each position to every other alphabet character.
4. Keep the mutation that gets closest to `2026`.
5. If stuck in a local optimum, randomly mutate a few characters and continue.
6. Repeat until the score is exactly `2026`.

This works because the score function is visible through the binary, and nearby profiles often produce meaningfully different scores.

## Solver Script

```python
#!/usr/bin/env python3
import random
import re
import subprocess
import sys
from functools import lru_cache

BIN = "./faultline"
ALPH = "BCDFGHJKLMNPQRST"
TARGET = 2026
N = 12

score_re = re.compile(r"(-?\d+)")


def run(*args: str) -> str:
    p = subprocess.run([BIN, *args], capture_output=True, text=True)
    return (p.stdout + p.stderr).strip()


@lru_cache(maxsize=None)
def score(profile: str) -> int:
    out = run("score", profile)
    m = score_re.search(out)
    if not m:
        raise RuntimeError(f"could not parse score from:\n{out}")
    return int(m.group(1))


def token(profile: str) -> str:
    out = run("token", profile).strip()
    return out.splitlines()[-1].strip()


def submit(profile: str, tok: str) -> str:
    return run("submit", profile, tok)


def random_profile() -> str:
    return "".join(random.choice(ALPH) for _ in range(N))


def all_single_mutations(profile: str):
    for i in range(N):
        for c in ALPH:
            if c == profile[i]:
                continue
            yield profile[:i] + c + profile[i + 1:]


def improve(profile: str):
    cur = score(profile)
    best_p, best_s = profile, cur
    best_d = abs(cur - TARGET)

    for cand in all_single_mutations(profile):
        s = score(cand)
        d = abs(s - TARGET)
        if d < best_d:
            best_p, best_s, best_d = cand, s, d

    return best_p, best_s


def search(restarts=200):
    global_best = None
    global_best_s = None

    for _ in range(restarts):
        p = random_profile()
        s = score(p)
        seen = set()

        for _ in range(100):
            if s == TARGET:
                return p, s

            state = (p, s)
            if state in seen:
                break
            seen.add(state)

            np, ns = improve(p)

            if np == p:
                # Local optimum: randomly perturb the profile.
                p = list(p)
                for _ in range(random.randint(1, 3)):
                    idx = random.randrange(N)
                    p[idx] = random.choice(ALPH)
                p = "".join(p)
                s = score(p)
            else:
                p, s = np, ns

            if global_best is None or abs(s - TARGET) < abs(global_best_s - TARGET):
                global_best, global_best_s = p, s
                print(f"[best] {global_best} -> {global_best_s}", file=sys.stderr)

    return global_best, global_best_s


if __name__ == "__main__":
    p, s = search()
    print(f"[+] best profile: {p}")
    print(f"[+] best score:   {s}")

    if s == TARGET:
        t = token(p)
        print(f"[+] token: {t}")
        print("[+] submit result:")
        print(submit(p, t))
    else:
        print("[-] exact 2026 not found yet; rerun or increase restarts")
```

## Running the Solver

```bash
$ python3 solve2.py
[best] QHJMRTPCQMTR -> -198
[best] QHMMRTPCQMTR -> 65
[best] QHMMRBPCQMTR -> 313
[best] QHMMRBPCQMPR -> 446
[best] QHMKRBPCQMPR -> 502
[best] DHMKRBPCQMPR -> 560
[best] GSKNJKFFGKKG -> 614
[best] HLKNLKCFGKKG -> 622
[best] TLKNLKCFGKKG -> 649
[best] QSLMKCFLJRKH -> 817
[best] FFGCLKRDNNTT -> 823
[best] FHGCLKRDNNTT -> 843
[best] KHGCLKRDNNTT -> 892
[best] FGPDDNGQSRLP -> 911
[best] FGPDDNGQSRLN -> 992
[best] JSFLQFMHSCPG -> 1020
[best] JSFLSFMHSCPG -> 1030
[best] SDPKGTCPJGTB -> 1144
[best] SDPKGTCPJFTB -> 1302
[best] SDPKGTCMGSCP -> 1420
[best] SDPKGTCMJSCP -> 1540
[best] SDPKGTCMJSCM -> 1659
[best] SDPKGTCMJRFL -> 2026
[+] best profile: SDPKGTCMJRFL
[+] best score:   2026
[+] token: Z2L-2F5-BUBP
[+] submit result:
CIT{12z4PXVTa3x3}
```

The solver found the profile:

```text
SDPKGTCMJRFL
```

Its score is exactly:

```text
2026
```

## Manual Verification

We can verify the score manually:

```bash
$ ./faultline score SDPKGTCMJRFL
2026
```

Then generate the token:

```bash
$ ./faultline token SDPKGTCMJRFL
Z2L-2F5-BUBP
```

Finally, submit the profile and token:

```bash
$ ./faultline submit SDPKGTCMJRFL Z2L-2F5-BUBP
CIT{12z4PXVTa3x3}
```

## Flag

```text
CIT{12z4PXVTa3x3}
```

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


