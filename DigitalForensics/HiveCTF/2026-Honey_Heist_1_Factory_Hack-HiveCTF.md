# Honey Heist I: Factory Hack Writeup

## Challenge

We are given a packet capture from a honey production facility and asked to figure out what the attacker did at the first site.

The challenge hints that this is an investigation into industrial/factory activity, so the first thing to check is whether the traffic contains industrial control protocols.

## Initial Analysis

Open the packet capture in Wireshark and look at the protocol list.

A suspicious protocol appears:

```text
Modbus/TCP
```

Modbus is commonly used for PLC and industrial control systems. It usually runs on TCP port `502`.

Filtering for Modbus traffic:

```wireshark
modbus
```

or:

```wireshark
tcp.port == 502
```

shows communication between:

```text
Attacker: 10.0.117.22
PLC:      10.0.117.14
Port:     502
```

## Suspicious Writes

The attacker sends Modbus write requests to the PLC.

Two important register writes appear first:

```text
Write register 0x0000 = 0x0096
Write register 0x0031 = 0x1f40
```

Convert the values from hexadecimal:

```text
0x0096 = 150
0x1f40 = 8000
```

So the attacker changed two PLC values:

```text
Register 0x0000 = 150
Register 0x0031 = 8000
```

Based on the factory context, these likely correspond to something like:

```text
temp_setpoint = 150
rpm_setpoint  = 8000
```

This means the attacker likely tried to overheat or overdrive part of the honey production process.

## Finding the Flag

After the dangerous register changes, the attacker writes more values to consecutive holding registers starting around address `0x00e8`.

The values look like ASCII characters.

Example:

```text
0x00e8 = 0x48 = H
0x00e9 = 0x69 = i
0x00ea = 0x76 = v
0x00eb = 0x65 = e
0x00ec = 0x43 = C
0x00ed = 0x54 = T
0x00ee = 0x46 = F
0x00ef = 0x7b = {
```

Decoding the bytes as ASCII gives:

```text
HiveCTF{0v3rh34t1ng_th3_h0n3y}
```

## Solution Script

A quick Python script can decode the register values if they are extracted from the packet capture:

```python
values = [
    0x48, 0x69, 0x76, 0x65, 0x43, 0x54, 0x46, 0x7b,
    0x30, 0x76, 0x33, 0x72, 0x68, 0x33, 0x34, 0x74,
    0x31, 0x6e, 0x67, 0x5f, 0x74, 0x68, 0x33, 0x5f,
    0x68, 0x30, 0x6e, 0x33, 0x79, 0x7d
]

flag = ''.join(chr(v) for v in values)
print(flag)
```

Output:

```text
HiveCTF{0v3rh34t1ng_th3_h0n3y}
```

## Flag

```text
HiveCTF{0v3rh34t1ng_th3_h0n3y}
```

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
