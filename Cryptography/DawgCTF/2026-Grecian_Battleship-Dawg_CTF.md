grecian battleship writeup.md :- # Ancient Battleship Writeup

## Challenge

We are given a Linux executable named `ancientbattleship` and the hint:

> Consider why the AI behaves so predictably..

Flag format:

```text
DawgCTF{}
```

## Initial Recon

The file is a PyInstaller-packed ELF executable rather than a normal native binary.

Useful checks:

```bash
file /media/sf_kali/ancientbattleship
strings -n 8 /media/sf_kali/ancientbattleship | grep -i pyi
pyinstxtractor /media/sf_kali/ancientbattleship
```

After extraction, the interesting file is:

```text
battleship.pyc
```

Decompiling or disassembling that bytecode reveals the core game logic.

## Important Code

The most important part is the AI move list:

```python
self.move_script = [
    (2, 4),
    (2, 3),
    (2, 1),
    (0, 0),
    (1, 1),
    (3, 1),
    (3, 4),
    (2, 2),
    (0, 4),
    (3, 3)
]
```

And the AI turn logic just consumes this list in order.

So the AI is "predictable" because it is not actually thinking. Its moves are hardcoded.

At first glance, this seems like the solve is simply to beat the AI by exhausting the script. That does work in-game, but it only gives:

```text
The AI is out of moves. You win!
```

That is not the flag.

## Interpreting the Hint

The challenge title and hint matter:

- `Ancient Battleship`
- `Ancient Greeks`
- a fixed sequence of coordinate pairs
- a `5x5` grid

That strongly suggests a **Polybius square**, an Ancient Greek cipher that uses 5x5 coordinates.

So instead of only using the move list as game logic, we should also treat it as encoded data.

## Extracting the Data

The scripted coordinates are:

```python
(2,4), (2,3), (2,1), (0,0), (1,1), (3,1), (3,4), (2,2), (0,4), (3,3)
```

Convert them from zero-based to one-based coordinates:

```text
35 34 32 11 22 42 45 33 15 44
```

Now decode them with a standard Polybius square:

```text
1 2 3 4 5
1 A B C D E
2 F G H I/J K
3 L M N O P
4 Q R S T U
5 V W X Y Z
```

Mapping each pair:

```text
35 -> P
34 -> O
32 -> M
11 -> A
22 -> G
42 -> R
45 -> U
33 -> N
15 -> E
44 -> T
```

This spells:

```text
POMAGRUNET
```

## Flag

```text
DawgCTF{POMAGRUNET}
```

## Short Solve Script

```python
coords = [(2,4), (2,3), (2,1), (0,0), (1,1), (3,1), (3,4), (2,2), (0,4), (3,3)]
square = {
    "11":"A", "12":"B", "13":"C", "14":"D", "15":"E",
    "21":"F", "22":"G", "23":"H", "24":"I", "25":"K",
    "31":"L", "32":"M", "33":"N", "34":"O", "35":"P",
    "41":"Q", "42":"R", "43":"S", "44":"T", "45":"U",
    "51":"V", "52":"W", "53":"X", "54":"Y", "55":"Z",
}

flag_text = "".join(square[f"{r+1}{c+1}"] for r, c in coords)
print(f"DawgCTF{{{flag_text}}}")
```

Output:

```text
DawgCTF{POMAGRUNET}
```

* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
