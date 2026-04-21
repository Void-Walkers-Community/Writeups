# Hens can type? — Writeup

**Category:** Forensics
**Flag:** `UDCTF{k3y_StR0K3E_1S_7he_wAy}`

---

## The Challenge

We got a file called `challenge1.pcapng`. This is a USB traffic capture file. The challenge says someone typed something important. We need to find what they typed.

---

## Step 1 — Open the File

We use `tshark` to look inside the file.

```bash
tshark -r challenge1.pcapng | head -30
```

We can see USB packets. There are two devices sending data:
- `2.3.1` — sends many packets
- `2.11.1` — sends fewer packets

---

## Step 2 — Look at Device 2.3.1

We extract the data from device `2.3.1`:

```bash
tshark -r challenge1.pcapng -Y "usb.src == \"2.3.1\"" \
  -T fields -e usb.capdata | grep -v "^$"
```

The data looks like this:

```
0000c61ecf2c0000
0000251fb02c0000
...
```

Each packet has 8 bytes. We parse them as X and Y coordinates. This looks like a **drawing tablet**.

We plot all the points with Python:

```python
import struct
import matplotlib.pyplot as plt

data = []
with open('usbdata.txt') as f:
    for line in f:
        raw = bytes.fromhex(line.strip())
        x = struct.unpack('<H', raw[2:4])[0]
        y = struct.unpack('<H', raw[4:6])[0]
        data.append((x, y))

# Split into strokes (pen lifts = large jumps)
strokes = []
current = [data[0]]
for i in range(1, len(data)):
    dx = abs(data[i][0] - data[i-1][0])
    dy = abs(data[i][1] - data[i-1][1])
    if (dx**2 + dy**2)**0.5 > 300:
        if len(current) > 1:
            strokes.append(current)
        current = [data[i]]
    else:
        current.append(data[i])

for stroke in strokes:
    xs = [p[0] for p in stroke]
    ys = [p[1] for p in stroke]
    plt.plot(xs, ys, 'k-', linewidth=2)

plt.gca().invert_yaxis()
plt.savefig('drawing.png')
```

The result is a drawing of a **chicken**. This is a decoy — it is not the answer.

---

## Step 3 — Look at Device 2.11.1

We extract data from the second device:

```bash
tshark -r challenge1.pcapng -Y "usb.src == \"2.11.1\"" \
  -T fields -e usb.capdata | grep -v "^$"
```

The data looks like this:

```
0200000000000000
0200180000000000
0200000000000000
0200070000000000
...
```

This is a **USB keyboard** (HID) format. Each packet has 8 bytes:

| Byte | Meaning |
|------|---------|
| 0 | Modifier key (`0x02` = Left Shift) |
| 1 | Reserved (always `0x00`) |
| 2 | Key code |
| 3–7 | Extra keys (usually `0x00`) |

A packet of `0000000000000000` means no key is pressed. We skip these.

---

## Step 4 — Decode the Keystrokes

We write a Python script to turn key codes into letters. We also check the modifier byte — if it is `0x02`, the Shift key is held down.

```python
keymap = {
    0x04: ('a','A'), 0x05: ('b','B'), 0x06: ('c','C'), 0x07: ('d','D'),
    0x08: ('e','E'), 0x09: ('f','F'), 0x0a: ('g','G'), 0x0b: ('h','H'),
    0x0c: ('i','I'), 0x0d: ('j','J'), 0x0e: ('k','K'), 0x0f: ('l','L'),
    0x10: ('m','M'), 0x11: ('n','N'), 0x12: ('o','O'), 0x13: ('p','P'),
    0x14: ('q','Q'), 0x15: ('r','R'), 0x16: ('s','S'), 0x17: ('t','T'),
    0x18: ('u','U'), 0x19: ('v','V'), 0x1a: ('w','W'), 0x1b: ('x','X'),
    0x1c: ('y','Y'), 0x1d: ('z','Z'),
    0x1e: ('1','!'), 0x1f: ('2','@'), 0x20: ('3','#'), 0x21: ('4','$'),
    0x22: ('5','%'), 0x23: ('6','^'), 0x24: ('7','&'), 0x25: ('8','*'),
    0x26: ('9','('), 0x27: ('0',')'),
    0x2d: ('-','_'), 0x2f: ('[','{'), 0x30: (']','}'),
    0x2c: (' ',' '), 0x28: ('\n','\n'), 0x2a: ('\b','\b'),
}

typed = ''
prev_key = 0

for line in kbd_data:
    raw = bytes.fromhex(line)
    modifier = raw[0]
    key = raw[2]
    shift = (modifier & 0x22) != 0

    if key == 0 or key == prev_key:
        prev_key = key
        continue
    prev_key = key

    if key in keymap:
        ch = keymap[key][1 if shift else 0]
        if ch == '\b':
            typed = typed[:-1]
        else:
            typed += ch

print(typed)
```

Output:

```
UDCTF{k3y_StR0K3E_1S_7he_wAy}
```

---

## Summary

The capture file had two USB devices:

1. **Drawing tablet (`2.3.1`)** — Someone drew a chicken. This was a trick to make us look at the wrong device.
2. **Keyboard (`2.11.1`)** — Someone typed the flag. This was the important device.

We decoded the keyboard HID packets by mapping each key code to a letter, and used the modifier byte to check for Shift.

The chicken was a decoy. The keyboard had the secret.

---

## Flag

```
UDCTF{k3y_StR0K3E_1S_7he_wAy}
```
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
