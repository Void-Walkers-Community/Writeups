# The Hive's Secret — Writeup

## Challenge

**Name:** The Hive's Secret  
**Category:** Reverse Engineering / Binary Analysis  
**Given hint:**

> Is it really just strings? Nah not quite, let's go in the opposite direction.

We are given a downloadable binary. The goal is to recover the hidden flag.

---

## Files

After downloading and extracting the challenge archive:

```bash
unzip The_Hives_Secret.zip
ls -la
file secret
```

Output:

```text
secret: ELF 64-bit LSB pie executable, x86-64, dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, not stripped
```

The binary is a 64-bit Linux ELF executable, and importantly, it is **not stripped**, meaning function names such as `main` and `check_flag` are still visible.

---

## First attempt: using `strings`

Since the challenge title and hint mention strings, the first obvious step is to run `strings`:

```bash
strings secret
```

Interesting output:

```text
HiveCTF{H
str1ngs_H
4r3_us3fH
t00l
===========================================
         Welcome 2 Strings Challenge       
Can you find what's hidden in this binary?
%s%s%s%s
HINT: Try running 'strings' on this binary
HINT: Look for something that starts with Hive
What am I hiding? Maybe: HiveCTF{
...you'll need to look harder than that :)
Enter the flag: 
[+] Correct! Well done!
[+] Flag accepted: %s
[-] Wrong flag. Keep digging!
```

At first glance, this looks like the flag may be stored directly in the binary. However, the chunks shown by `strings` are suspicious:

```text
HiveCTF{H
str1ngs_H
4r3_us3fH
t00l
```

They are not enough to form a valid flag. Some chunks contain extra `H` characters, and the ending is incomplete.

The hint says:

> Nah not quite, let's go in the opposite direction.

So instead of relying only on `strings`, we need to reverse the binary.

---

## Checking symbols

Because the binary is not stripped, we can list symbols with `nm`:

```bash
nm -C secret | grep -E 'main|check|flag'
```

Output:

```text
0000000000001270 T check_flag
000000000000134f T main
```

There is a function called `check_flag`, which is likely responsible for constructing or comparing the correct flag.

---

## Disassembling `check_flag`

Use `objdump` to inspect the function:

```bash
objdump -d -M intel secret | sed -n '/<check_flag>:/,/^$/p'
```

Important part of the disassembly:

```asm
0000000000001270 <check_flag>:
    ...
    1295: 48 b8 48 69 76 65 43 54 46 7b
    129f: 48 89 45 92

    12a7: 48 b8 73 74 72 31 6e 67 73 5f
    12b1: 48 89 45 9b

    12b9: 48 b8 34 72 33 5f 75 73 33 66
    12c3: 48 89 45 a4
    12c7: c7 45 ac 75 6c 5f 00

    12ce: c7 45 8b 74 30 30 6c
    12d5: c7 45 8e 6c 73 7d 00

    ...
    12fe: lea rdx,[rip+0xd8f]  ; "%s%s%s%s"
    1312: call snprintf@plt
    ...
    132c: call strcmp@plt
```

This tells us that the program is building the real flag at runtime using `snprintf` and then comparing our input using `strcmp`.

---

## Understanding little endian storage

The bytes in the instructions are stored in little endian order, but when interpreted as strings in memory, they become readable ASCII.

For example:

```asm
48 b8 48 69 76 65 43 54 46 7b
```

The immediate value represents these bytes:

```text
48 69 76 65 43 54 46 7b
```

Converting from hex to ASCII:

```text
H i v e C T F {
```

So the first chunk is:

```text
HiveCTF{
```

Doing this for the other chunks gives:

```text
str1ngs_
4r3_us3ful_
t00ls}
```

The program then combines them using:

```c
snprintf(buffer, 0x40, "%s%s%s%s", part1, part2, part3, part4);
```

So the final flag is built as:

```text
HiveCTF{ + str1ngs_ + 4r3_us3ful_ + t00ls}
```

---

## Verifying dynamically

We can run the binary and submit the reconstructed flag:

```bash
./secret
```

Input:

```text
HiveCTF{str1ngs_4r3_us3ful_t00ls}
```

Output:

```text
[+] Correct! Well done!
[+] Flag accepted: HiveCTF{str1ngs_4r3_us3ful_t00ls}
```

---

## Flag

```text
HiveCTF{str1ngs_4r3_us3ful_t00ls}
```

---

## Summary

The challenge tries to mislead us into relying only on `strings`. While `strings` reveals some useful hints and partial flag-looking chunks, the actual flag is constructed at runtime inside `check_flag`. By disassembling the binary and inspecting the calls to `snprintf` and `strcmp`, we can recover the correct order and contents of the flag chunks.

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
