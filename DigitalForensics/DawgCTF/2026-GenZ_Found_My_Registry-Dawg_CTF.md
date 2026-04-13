# DawgCTF Registry Challenge Writeup

## Summary

We are given a single file:

- `/media/sf_kali/chal.reg`

The challenge asks us to find all registry changes and recover the flag.

The correct flag is:

`DawgCTF{qu33n_0f_th3_h1v3}`

## Initial Clue

The hint says:

`H1: What is a powerful tool that might be useful to look at a registry?`

A strong first step is using `strings` on the `.reg` export, or otherwise reading the registry file as text. Since `chal.reg` is a UTF-16LE registry export, converting it to UTF-8 also helps:

```bash
iconv -f UTF-16LE -t UTF-8 /media/sf_kali/chal.reg | less
```

## Suspicious Entries

At the very top of the registry export, two obviously fake service entries appear:

```reg
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\+7]
...
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\+7\Parameters]
"evens"=""

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\-6]
...
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\-6\Parameters]
"odds"=""
```

These do not look like legitimate service names. They are clearly challenge clues:

- `+7`
- `-6`
- `evens`
- `odds`

So the file is telling us a string must be split into odd/even positions and decoded with shifts of `+7` and `-6`.

## Finding the Hidden Payload

Searching the file for unusual short string values reveals many one-character entries whose names are numbers:

```text
"1"="J"
"2"="Z"
"3"="}"
"4"="`"
...
"26"="v"
```

These are scattered across different service keys inside the same `chal.reg` file.

If we collect them in numeric order from `1` to `26`, we get:

```text
JZ}`IMLtwn9,tX6_emn,ea7o9v
```

## Decoding

Now apply the clue:

1. Split the string into odd and even positions.
2. Decode the odd-position characters with `-6`.
3. Decode the even-position characters with `+7`.
4. Interleave them back together.

Encoded string:

```text
JZ}`IMLtwn9,tX6_emn,ea7o9v
```

Odd-position characters:

```text
J}ILw9t6ene79
```

Even-position characters:

```text
Z`Mtn,X_m,aov
```

Decode:

- odd chars minus 6 -> `DwCFq3n0_h_13`
- even chars plus 7 -> `agT{u3_ft3hv}`

Interleaving those decoded halves gives:

```text
DawgCTF{qu33n_0f_th3_h1v3}
```

## Reproduction Script

This short Python snippet reproduces the decode:

```python
enc = "JZ}`IMLtwn9,tX6_emn,ea7o9v"

odd = enc[::2]
even = enc[1::2]

odd_dec = ''.join(chr(ord(c) - 6) for c in odd)
even_dec = ''.join(chr(ord(c) + 7) for c in even)

flag = ''.join(a + b for a, b in zip(odd_dec, even_dec))
print(flag)
```

Output:

```text
DawgCTF{qu33n_0f_th3_h1v3}
```

## Final Answer

`DawgCTF{qu33n_0f_th3_h1v3}`

* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

