# Tunnel Write-up

## Challenge
Our information security department detected suspicious activity on a workstation.  
It appeared that an attacker exfiltrated data via a non-standard channel.

Flag format: `KubSTU{}`

## 1. Initial Triage

Inspect the capture:

```bash
capinfos /media/sf_kali/Krasnodar.pcap
```

Key points:
- ~80k packets
- Duration: ~37 seconds
- Mostly one-way synthetic traffic from several internal hosts

Protocol hierarchy:

```bash
tshark -r /media/sf_kali/Krasnodar.pcap -q -z io,phs
```

Conversations:

```bash
tshark -r /media/sf_kali/Krasnodar.pcap -q -z conv,ip
```

One host stood out:
- `192.168.1.50 -> 8.8.4.4` (small, unique flow compared to noisy traffic)

## 2. Isolate Suspicious Host

Filter packets from the suspicious source:

```bash
tshark -r /media/sf_kali/Krasnodar.pcap -Y "ip.src==192.168.1.50" -T fields -e _ws.col.Protocol | sort | uniq -c
```

Result:
- All 540 packets are `DNS` queries.

Extract queried domains:

```bash
tshark -r /media/sf_kali/Krasnodar.pcap \
  -Y "ip.src==192.168.1.50 && dns.flags.response==0" \
  -T fields -e dns.qry.name > /tmp/kras_dns_qnames.txt
```

All queries are to:
- `*.exfiltrate.kubstu-ctf.ru`

This is explicit DNS tunneling/exfiltration.

## 3. Recover Encoded Payload

Look for structure in labels:

```bash
grep -n '^v[0-9][0-9]\.' /tmp/kras_dns_qnames.txt
```

Markers found:
- `v00.4b75`
- `v01.6253`
- `v02.5455`
- ...
- `v20.787d`

The second label is hex. Concatenate in order:

```text
4b75 6253 5455 7b64 306e 745f 7472 7535 745f 7468 335f 646e 355f 7175 3372 3133 355f 7631 615f 6833 787d
```

Joined:

```text
4b75625354557b64306e745f74727535745f7468335f646e355f717533723133355f7631615f6833787d
```

Decode hex:

```bash
echo '4b75625354557b64306e745f74727535745f7468335f646e355f717533723133355f7631615f6833787d' | xxd -r -p
```

## 4. Flag

`KubSTU{d0nt_tru5t_th3_dn5_qu3r135_v1a_h3x}`


---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

