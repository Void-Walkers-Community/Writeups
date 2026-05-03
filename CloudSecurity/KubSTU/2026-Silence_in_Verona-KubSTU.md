# Silence in Veronna - Writeup

## Challenge
- File: `/media/sf_kali/3_in_1_.pkt`
- Flag format: `KubSTU(...)`

## 1) Decrypt the `.pkt`
Used Unpacket to recover XML:

```bash
python3 /home/parth/HACK/Unpacket/unpacket.py /media/sf_kali/3_in_1_.pkt -o /home/parth/HACK/3_in_1_.xml
```

Result: `/home/parth/HACK/3_in_1_.xml`

## 2) Initial triage (decoys and infra clues)
From XML/config/logs:
- Decoy interface description contains fake flag:
  - `KubSTU(gazebo_is_stronger_than_tarask)`
- Real network secret appears in hash/banner:
  - `d20_SaltoNazad`
- Port/VLAN/topology clues are present, but they do not directly produce accepted flag.

So the direct network strings are mostly misdirection.

## 3) Hidden payload found in workstation files
A file named:
- `Borring novel .txt`

Its content is not normal prose; it is Shakespeare Programming Language (SPL)-style code (lines like `Romeo: ... Speak your mind!`).

Extracted it to:
- `/home/parth/HACK/borring_novel.spl`

## 4) Execute the Shakespeare program
Loaded local `horatio` interpreter source and executed the SPL program via Node.

Important syntax fix for this interpreter:
- Convert stage directions using `and` into `&`, e.g.:
  - `[Enter Romeo and Juliet]` -> `[Enter Romeo & Juliet]`

Program output:

```text
Good job bookworm, here's your reward
Mellin_The_Hunter
```

## 5) Final flag

```text
KubSTU(Mellin_The_Hunter)
```




---
* [🔙 Back to Cloud Security Directory](../)
* [🔙 Back to Cloud Security Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
