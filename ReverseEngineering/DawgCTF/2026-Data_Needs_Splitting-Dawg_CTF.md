# DawgCTF - data-needs-splitting

## Flag

`DawgCTF{J@v@_My_B3l0v3d}`

## Summary

The domain `data-needs-splitting.umbccd.net` does not serve the challenge over HTTP.  
The hint to keep using `dig` points to DNS as the transport layer.

The binary is stored in split DNS `TXT` records, reconstructed into a JAR, and then reversed to recover the flag.

## 1. Discover how the data is hosted

Trying to open the domain in a browser or with `curl` does not help because there is no webserver involved.

Checking DNS with `dig` is the key step:

```bash
dig @1.1.1.1 data-needs-splitting.umbccd.net TXT +short
```

This returns multiple TXT records that look like:

```text
"00UEsDBAoAAAgAANu0h1w..."
"01QAAABQSwMEFAAICAgA..."
"02xiWpAjxDRyyeO5I0pe..."
...
"16hqwnuawCAADuBAAACg..."
```

Each record starts with a two-digit index, followed by base64 data.

The `UEsD...` prefix is a strong clue that the reconstructed file is a ZIP/JAR archive.

## 2. Rebuild the challenge binary

Sort the chunks, strip the numeric prefixes, concatenate them, and base64-decode:

```bash
python3 - <<'PY'
import subprocess, shlex

out = subprocess.check_output(
    shlex.split("dig @1.1.1.1 data-needs-splitting.umbccd.net TXT +short"),
    text=True,
)

parts = []
for line in out.splitlines():
    line = line.strip()
    if line.startswith('"') and line.endswith('"'):
        line = line[1:-1]
    parts.append(line)

parts.sort()
b64 = ''.join(p[2:] for p in parts)

open("dns_payload.b64", "w").write(b64)
PY

base64 -d dns_payload.b64 > challenge.jar
unzip -o challenge.jar -d dns_extract
```

Recovered files:

```text
dns_extract/Main.class
dns_extract/Loader.class
dns_extract/assets/file.dat
dns_extract/META-INF/MANIFEST.MF
```

## 3. Reverse the JAR

Decompile the visible classes:

```bash
javap -classpath dns_extract -c -p Main Loader
```

`Main` does three important things:

1. Creates a `Loader`
2. Loads `/assets/file.dat` as a class
3. Instantiates it and calls `validate()`

Relevant logic:

```java
Loader l = new Loader();
Class<?> c = l.load("/assets/file.dat");
Object o = c.getDeclaredConstructor().newInstance();
boolean ok = (Boolean) c.getMethod("validate").invoke(o);
```

That means `assets/file.dat` is actually a hidden Java class.

Rename it and decompile it:

```bash
cp dns_extract/assets/file.dat dns_extract/Validator.class
javap -classpath dns_extract -c -p Validator
```

## 4. Understand the validator

The validator:

1. Prompts for the flag
2. Reads the input
3. XORs each character with a repeating 4-position mask
4. Appends the resulting integers as decimal text
5. Compares that long string to a constant

Important constants from the bytecode:

```java
long a = 2194307438957234483L;
long b = 148527584754938272L;
String target = "145511939249997195145441944550467175145531942549987228145401943650017203145451934650207244145651934650127169";
```

For each position `i`, the code does:

```java
mask1 = (char)((a >> ((i % 4) * 16)) & 0xffff);
mask2 = (char)((b >> ((i % 4) * 16)) & 0xffff);
value = input[i] ^ mask1 ^ mask2;
```

So the effective repeating 4-value mask is:

```text
[14483, 19361, 5104, 7292]
```

## 5. Recover the flag

Because the flag format is known to be `DawgCTF{...}`, we can solve the decimal-string partitioning directly.

Solver:

```bash
python3 - <<'PY'
from functools import lru_cache

A = 2194307438957234483
B = 148527584754938272
target = "145511939249997195145441944550467175145531942549987228145401943650017203145451934650207244145651934650127169"

masks = [((A >> (16 * i)) & 0xffff) ^ ((B >> (16 * i)) & 0xffff) for i in range(4)]
allowed = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_{}!@#$%^&*()-=+[]:;,.?/"

@lru_cache(None)
def solve(pos, idx):
    if pos == len(target):
        return ""
    for ch in allowed:
        s = str(ord(ch) ^ masks[idx % 4])
        if target.startswith(s, pos):
            rest = solve(pos + len(s), idx + 1)
            if rest is not None:
                return ch + rest
    return None

print(solve(0, 0))
PY
```

Output:

```text
DawgCTF{J@v@_My_B3l0v3d}
```

## Final Answer

`DawgCTF{J@v@_My_B3l0v3d}`


---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

