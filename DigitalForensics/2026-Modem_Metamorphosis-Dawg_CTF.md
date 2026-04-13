i love bacon! :-
# DNS C2 Writeup

## Challenge Summary

We are given a single packet capture, `dns_c2.pcap`, and asked to identify suspicious DNS traffic and recover the flag.

## Initial Triage

The capture contains repetitive DNS `A` queries from `10.67.0.2` to `10.1.1.53`.

The suspicious pattern is:

- 1000 unique DNS queries
- all under `dawg.cwa.sec`
- high-entropy uppercase subdomains
- no meaningful DNS answers returned

That is classic DNS tunneling / DNS C2 behavior, where data is hidden in the query names themselves.

## Extracting The Query Names

Use `tshark` to pull the DNS requests:

```bash
tshark -r /media/sf_kali/dns_c2.pcap \
  -Y 'dns.flags.response == 0' \
  -T fields -e frame.number -e dns.qry.name
```

Example suspicious packets:

- `533` -> `IRQXOZ2DKRDHW4ZRPJ5GY2LO.dawg.cwa.sec`
- `909` -> `L5ZXKY3DOVWDG3TU.dawg.cwa.sec`
- `1823` -> `L5RTEX3CGRRW63T5.dawg.cwa.sec`

## Decoding

The leftmost DNS label is base32-like. Strip `.dawg.cwa.sec`, then base32-decode the encoded label/stream to recover embedded text.

A minimal script:

```python
import base64
import re
import subprocess

out = subprocess.check_output(
    "tshark -r /media/sf_kali/dns_c2.pcap "
    "-Y 'dns.flags.response == 0' "
    "-T fields -e frame.number -e dns.qry.name",
    shell=True,
    text=True,
)

for line in out.splitlines():
    frame, name = line.split("\t", 1)
    label = re.sub(r"\.dawg\.cwa\.sec$", "", name)
    pad = "=" * ((8 - len(label) % 8) % 8)
    try:
        decoded = base64.b32decode(label + pad, casefold=True)
    except Exception:
        continue

    printable = "".join(chr(c) if 32 <= c < 127 else "." for c in decoded)
    if "DawgCTF" in printable or "succul3nt" in printable or "b4con" in printable:
        print(frame, printable)
```

## Recovered Flag Fragments

Relevant packets expose the flag text in pieces:

- Frame `533` -> `DawgCTF{s1zzlin`
- Frame `909` -> `_succul3nt`
- Frame `1823` -> `_c2_b4con}`

Combining the recovered fragments yields:

```text
DawgCTF{s1zzlin_succul3nt_c2_b4con}
```

## Flag

```text
DawgCTF{s1zzlin_succul3nt_c2_b4con}
```

* [🔙 Back to Digital Forensics Directory](../DigitalForensics)
* [🔙 Back to Digital Forensics Index Directory](../DigitalForensics/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
