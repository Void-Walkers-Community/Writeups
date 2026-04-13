# The Step After the Pcap

## Challenge Summary
We were given the output of an LLM-based network flow analyzer over a packet capture. The report itself already hinted at two important problems:

1. **The timestamps were out of order**.
2. **The analyzer had not properly identified the destination of interest**.

The task was to recover the payload fragments, place them in true chronological order, and join them with underscores.

---

## Initial Observations
At the top of the report, the analyzer states:

- a repeated TLS JA3 hash was seen across multiple flows to the same IP address
- the times seemed to be out of order

That immediately suggests the useful traffic is not grouped by flow record number, but instead should be pivoted by:

- common destination IP
- common JA3 hash
- non-empty payload fragments
- actual timestamp order

---

## Pivoting to the Suspicious Traffic
Reviewing the flow records shows a clear cluster of TLS sessions with:

- **Destination IP:** `45.76.123.45`
- **Protocol:** `TLS`
- **Destination Port:** `443`
- **JA3 hash:** `d2b4c6a8f0e1d3c5b7a9f2e4d6c8b0a1`

These are the only records that consistently carry meaningful `Payload Fragment` values rather than `-`.

So the correct approach is:

1. Filter all flow records where `Dst IP = 45.76.123.45`
2. Keep the records whose `Payload Fragment` is not `-`
3. Sort them by `Timestamp`
4. Join the fragments with underscores

---

## Extracted Fragments in Chronological Order

| Timestamp (UTC) | Payload Fragment |
|---|---|
| 2026-04-05 00:12:10 | HBRPO |
| 2026-04-05 01:25:15 | IG8F1 |
| 2026-04-05 01:28:28 | CBFNO |
| 2026-04-05 01:36:50 | 6B9M8 |
| 2026-04-05 01:50:04 | 0O2RA |
| 2026-04-05 01:57:33 | K1VRJ |
| 2026-04-05 02:23:49 | NVGFY |
| 2026-04-05 03:03:19 | GWWQC |
| 2026-04-05 03:12:15 | 38HYF |
| 2026-04-05 04:29:15 | 9SXME |
| 2026-04-05 04:51:18 | COSFO |
| 2026-04-05 05:17:55 | GYR3X |
| 2026-04-05 05:24:41 | KXWNR |
| 2026-04-05 05:26:29 | EK8PK |
| 2026-04-05 05:53:07 | 3YR9O |
| 2026-04-05 06:06:47 | UDOCU |
| 2026-04-05 06:18:19 | ZRENU |
| 2026-04-05 06:34:19 | N5Z3J |
| 2026-04-05 06:36:37 | QIP98 |
| 2026-04-05 07:25:16 | Q1ZXO |
| 2026-04-05 07:28:12 | I65FD |
| 2026-04-05 07:36:53 | HJK1E |
| 2026-04-05 07:54:06 | YY37Q |
| 2026-04-05 07:54:55 | 9AH8R |
| 2026-04-05 08:00:57 | VHS1K |
| 2026-04-05 08:08:09 | 3AQ6L |
| 2026-04-05 08:12:38 | 6GT6M |
| 2026-04-05 09:02:56 | JXK87 |
| 2026-04-05 09:10:53 | AU5BH |
| 2026-04-05 09:11:15 | XTPDP |
| 2026-04-05 09:13:12 | FF5E8 |
| 2026-04-05 09:36:03 | II49K |
| 2026-04-05 09:51:25 | Q71N8 |
| 2026-04-05 09:56:27 | MTZX2 |
| 2026-04-05 10:07:04 | 72HPO |
| 2026-04-05 10:50:40 | EVB9O |
| 2026-04-05 11:02:03 | OAEDO |
| 2026-04-05 11:14:45 | ECVE6 |
| 2026-04-05 11:40:16 | PR5N8 |
| 2026-04-05 11:51:14 | I4P40 |
| 2026-04-05 12:34:12 | MGG1W1 |

---

## Final Payload String

```text
HBRPO_IG8F1_CBFNO_6B9M8_0O2RA_K1VRJ_NVGFY_GWWQC_38HYF_9SXME_COSFO_GYR3X_KXWNR_EK8PK_3YR9O_UDOCU_ZRENU_N5Z3J_QIP98_Q1ZXO_I65FD_HJK1E_YY37Q_9AH8R_VHS1K_3AQ6L_6GT6M_JXK87_AU5BH_XTPDP_FF5E8_II49K_Q71N8_MTZX2_72HPO_EVB9O_OAEDO_ECVE6_PR5N8_I4P40_MGG1W1
```

---

## Conclusion
The intended trick was not to trust the analyzer's flow ordering. Instead, the correct solution came from identifying the repeated TLS beaconing pattern:

- same destination IP
- same JA3 fingerprint
- fragmented payloads spread across many sessions
- timestamps requiring manual chronological reconstruction

Once those records were sorted by timestamp, the final payload string could be rebuilt reliably.



* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
