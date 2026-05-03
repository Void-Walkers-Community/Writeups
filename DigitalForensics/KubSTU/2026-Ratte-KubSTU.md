# RATTE PCAP Incident Response Writeup

## Scenario
We received a suspicious packet capture: `/media/sf_kali/Ratte.pcap`.

Goal: identify what is wrong in the traffic and recover the flag in format `KubSTU{...}`.

## Initial Triage
Basic file metadata:

- Packets: `1227`
- Duration: `0.236312s`
- Encapsulation: `Raw IPv4`
- Capture window: `2026-03-21 14:19:50.008736` to `2026-03-21 14:19:50.245048` (IST)

This is a very short, high-density burst, typical of scripted or staged malicious traffic.

## Protocol and Conversation Analysis
Protocol hierarchy showed:

- `udp` + `dns` heavy noise
- `tcp` with mixed `http/tls/ftp/ssh`

Conversation analysis revealed many one-packet flows designed to look busy.  
One flow stood out as persistent and structured:

- `10.0.0.5:49152 -> 10.0.0.15:1337`
- 21 packets total
- 20 packets with 5-byte TCP payload
- 1 final packet with 4-byte payload

This is the only meaningful continuous C2-like stream in the capture.

## Suspicious Stream Extraction
Using `tshark` on `tcp.stream == 102`, payloads were:

```text
deadbeef42
cc53020937
ccb9022011
cc75021617
cccb02392c
ccb102721d
cc61022f72
cc8e023071
cc00021d25
ccc8023071
cc40023232
cc6502732c
cc3b02251d
ccda02732c
cc2a021d36
cc45022a71
cc21021d26
cc45027630
cc2602291d
cc3f023470
cc1e013f
```

Pattern:

- First frame: marker/key-like value `deadbeef42`
- Subsequent frames: `cc ?? 02 xx yy`
- Final frame: `cc ?? 01 zz`

The byte `0x42` from `deadbeef42` is the XOR key.

## Decoding
Collecting the data bytes (`xx yy ... zz`) and XOR with `0x42` reveals:

```text
KubSTU{n0_m0r3_gr3pp1ng_1n_th3_d4rk_v2}
```

## What Is Wrong Here
The traffic is intentionally obfuscated:

- Bulk decoy traffic and repetitive DNS noise (`google.com`) to distract analysts.
- Real command/exfil channel hidden in a compact custom protocol on TCP `1337`.
- Payload encoded with simple XOR to avoid plain-text detection.

This is consistent with RAT/backdoor beacon or staged C2 telemetry behavior.

## Flag
`KubSTU{n0_m0r3_gr3pp1ng_1n_th3_d4rk_v2}`

## Key IOCs
- Source: `10.0.0.5`
- Destination C2-like endpoint: `10.0.0.15:1337`
- Stream index: `102`
- Obfuscation marker/key carrier: `deadbeef42`
- Framing byte prefix: `cc`

## Suggested Response Actions
1. Isolate host `10.0.0.5` immediately.
2. Block and monitor traffic to `10.0.0.15:1337`.
3. Hunt for the same framing pattern (`cc ?? 02`, `cc ?? 01`) across PCAP/NetFlow.
4. Perform host forensic triage on `10.0.0.5` for RAT persistence and dropped binaries.

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

