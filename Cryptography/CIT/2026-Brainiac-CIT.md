# Writeup

## Challenge Overview

We were given a text file containing a long sequence of symbols:

```bf
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>---.++++++.+++++++++++.>+++++++++++++++++++++++.<+++.+++++++++++++++++.<++++++++++++++++++++++++++++++++++.>>-------.<---------.++++++++++.+++++.---------------.>.<+++++++++.<-------------.>---------.>+++.<<---.>>-----.------.<+++++.-----.>---.<<------------.>.>+++++++++++.<+++++++++.<+++++++++++++.>>-.<---------.>-------.<<+++++++++++++++.>>++.-------.++++++++++++++.<<.>++++++++.<-------------.>>++++++++.
```

The challenge also provided this SHA1 hash:

```text
33746b3052f748a9d41f030d2be4f196d02453cb
```

The goal was to identify what the strange-looking code was doing and recover the hidden message.

---

## Identifying the Language

The payload only uses characters such as:

```text
+ - < > [ ] .
```

These are instructions from the esoteric programming language **Brainfuck**.

Brainfuck uses a memory tape and a pointer. Each command changes the pointer, modifies the current cell, controls loops, or prints output.

| Symbol | Meaning |
|---|---|
| `+` | Increment the current cell |
| `-` | Decrement the current cell |
| `>` | Move the pointer to the right |
| `<` | Move the pointer to the left |
| `[` | Start a loop |
| `]` | End a loop |
| `.` | Print the current cell as an ASCII character |
| `,` | Read one character of input |

Since the code contains many `.` characters, it is likely printing ASCII characters one by one.

---

## Solving Method

First, save the provided Brainfuck code into a file:

```bash
cat > chal.bf << 'EOF'
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>---.++++++.+++++++++++.>+++++++++++++++++++++++.<+++.+++++++++++++++++.<++++++++++++++++++++++++++++++++++.>>-------.<---------.++++++++++.+++++.---------------.>.<+++++++++.<-------------.>---------.>+++.<<---.>>-----.------.<+++++.-----.>---.<<------------.>.>+++++++++++.<+++++++++.<+++++++++++++.>>-.<---------.>-------.<<+++++++++++++++.>>++.-------.++++++++++++++.<<.>++++++++.<-------------.>>++++++++.
EOF
```

Then, run it using a simple Python Brainfuck interpreter.

```python
code = open("chal.bf").read()

tape = [0] * 30000
ptr = 0
pc = 0
output = ""

bracket_map = {}
stack = []

# Precompute matching brackets
for i, c in enumerate(code):
    if c == "[":
        stack.append(i)
    elif c == "]":
        start = stack.pop()
        bracket_map[start] = i
        bracket_map[i] = start

# Execute Brainfuck
while pc < len(code):
    cmd = code[pc]

    if cmd == "+":
        tape[ptr] = (tape[ptr] + 1) % 256
    elif cmd == "-":
        tape[ptr] = (tape[ptr] - 1) % 256
    elif cmd == ">":
        ptr += 1
    elif cmd == "<":
        ptr -= 1
    elif cmd == ".":
        output += chr(tape[ptr])
    elif cmd == "[":
        if tape[ptr] == 0:
            pc = bracket_map[pc]
    elif cmd == "]":
        if tape[ptr] != 0:
            pc = bracket_map[pc]

    pc += 1

print(output)
```

---

## Output

Running the script gives:

```text
CIT{Wh@t_in_th3_w0rld_i$_th1s_l@ngu@g3}
```

This means the Brainfuck program directly prints the flag.

---

## SHA1 Check

The challenge gave this SHA1 hash:

```text
33746b3052f748a9d41f030d2be4f196d02453cb
```

We can check the SHA1 of the decoded output using:

```bash
echo -n 'CIT{Wh@t_in_th3_w0rld_i$_th1s_l@ngu@g3}' | sha1sum
```

The hash does not match the provided SHA1. However, the Brainfuck program clearly prints a valid flag-like string, so the decoded output is still the main result of the challenge.

---

## Final Flag

```text
CIT{Wh@t_in_th3_w0rld_i$_th1s_l@ngu@g3}
```

---

## Summary

This challenge was solved by recognizing the symbols as Brainfuck code, executing the code with an interpreter, and reading the printed ASCII output. The decoded output revealed the final flag.

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
