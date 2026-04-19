# Hive Party Pack Writeup

## Challenge

**Name:** Hive Party Pack  
**Category:** Reverse Engineering
**Flag:** `HiveCTF{g01ng_b4ck_2_th3_cl4ss1cs}`

## Summary

The challenge gives a Wii/GameCube-style executable, `hive_party_pack.dol`. The description hints that there is a hidden game and mentions Kazuhisa Hashimoto, the creator of the Konami Code. Running the game in Dolphin and entering the classic Konami Code unlocks the hidden content, which reveals the flag.

```text
UP UP DOWN DOWN LEFT RIGHT LEFT RIGHT B A
```

## Files

After downloading and extracting the archive:

```bash
unzip Hive_Party_Pack.zip
ls -lah
```

The archive contains one file:

```text
hive_party_pack.dol
```

A `.dol` file is a Nintendo GameCube/Wii executable format. Some Linux tools may misidentify it because it is not an ELF binary.

## Initial Static Analysis

First, I checked the file and extracted readable strings:

```bash
file hive_party_pack.dol
strings -a -n 4 hive_party_pack.dol | head
strings -a -n 4 hive_party_pack.dol | grep -Ei 'honey|game|up|down|left|right|waggle|flag|vault|recipe'
```

Interesting strings appeared in the binary:

```text
The Buzziest Games on Wii!
Select a game:
Honey Flow
Guide honey through pipes!
Learn the waggle dance!
WAGGLE UP
WAGGLE DOWN
WAGGLE LEFT
WAGGLE RIGHT
```

This confirmed that the file was a small Wii-style party game with multiple mini-games. However, the flag was not visible in plaintext:

```bash
grep -a 'HiveCTF' hive_party_pack.dol
```

No direct flag string was found, so the flag was likely hidden behind a runtime unlock condition or encrypted/obfuscated data.

## Hint Interpretation

The challenge description says:

> games were better back when guys like Kazuhisa Hashimoto were making them

Kazuhisa Hashimoto is strongly associated with the **Konami Code**, one of the most famous cheat codes in video game history:

```text
UP UP DOWN DOWN LEFT RIGHT LEFT RIGHT B A
```

Since the challenge is a retro-style game and specifically mentions a game developer tied to cheat codes, this is the intended hint.

## Dynamic Analysis in Dolphin

The note says the game expects:

- the latest Dolphin emulator
- a sideways Wii Remote

So I configured Dolphin with a sideways Wii Remote layout and mapped the buttons carefully:

```text
D-pad Up     -> Up
D-pad Down   -> Down
D-pad Left   -> Left
D-pad Right  -> Right
Button 1/2 or mapped B/A depending on Dolphin layout
```

The important sequence is:

```text
UP UP DOWN DOWN LEFT RIGHT LEFT RIGHT B A
```

After entering the sequence in the game/menu, the hidden content unlocks.

## Why This Works

The game appears to track directional/button input and compare it against a hidden unlock sequence. The challenge text gives the clue directly through Kazuhisa Hashimoto, pointing to the Konami Code.

Because the flag is not stored plainly in the binary, simply running `strings` is not enough. The correct input path is needed to trigger the hidden game/flag reveal.

## Flag

```text
HiveCTF{g01ng_b4ck_2_th3_cl4ss1cs}
```
---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
