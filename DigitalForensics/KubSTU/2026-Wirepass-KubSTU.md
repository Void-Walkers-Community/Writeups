# Arctic Intelligence — Network Forensics Writeup

## Challenge
We were given a packet capture (`challenge.pcap`) and told that documents were transferred over a custom encrypted protocol between two nodes. Flag format: `KubSTU{...}`.

## 1. Initial Triage
- File: `/media/sf_kali/challenge.pcap`
- Main hosts: `172.20.0.2` and `172.20.0.3`
- Traffic contained many decoy protocols/services (HTTP, FTP, TLS, DNS, etc.)

Using conversation stats (`tshark -z conv,tcp`), several suspicious custom ports appeared:
- `7777`, `54321`, `12345`, `31337`, `4444`, `5555`

## 2. Identify the Real Custom Channel
Among suspicious streams, `tcp.stream 86` (`52936 -> 31337`) stood out:
- Payload started with ASCII `XFER` (`58 46 45 52`)
- Ended with `RECV:OK`
- Looked like a file transfer frame

Extracted raw stream and decoded structure:
- `bytes[0:4]` = `XFER`
- `bytes[4:20]` = 16-byte key/marker
- `bytes[20:24]` = big-endian length
- `bytes[24:...]` = encrypted payload

## 3. Decrypt the Payload
XOR-ing payload with repeating 16-byte key from `bytes[4:20]` produced valid data.

Result began with:
- `PK\x03\x04` (ZIP header)
- Filenames visible inside archive:
  - `mission_report.txt`
  - `roster.txt`
  - `map.txt`

So decrypted payload was a password-protected ZIP archive.

## 4. Recover ZIP Password
From full PCAP string extraction, a critical custom-protocol artifact appeared:
- `PASS:IcyFl1pp3r$2026`
- `ACK:OK`

Used password with 7z:
- `7z t -p'IcyFl1pp3r$2026' archive.zip` -> `Everything is Ok`

## 5. Extract Flag
After extraction, `mission_report.txt` contained:

`СЕКРЕТНЫЙ КОД ОПЕРАЦИИ: KubSTU{p1ngu1n_0p_k4p1b4r0v5k_f4ll5}`

## Flag
`KubSTU{p1ngu1n_0p_k4p1b4r0v5k_f4ll5}`

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

