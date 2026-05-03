# Is that report in english — Writeup

## Challenge Summary
We are given a suspicious PDF file:

- `/media/sf_kali/KUBSTU_Financial_Report_2025.pdf`
- Flag format: `KubSTU{...}`

The PDF looks like a normal 2-page financial report, but it contains hidden data and decoy flags.

## 1) Initial Recon
I checked basic metadata and extracted visible text:

- `pdfinfo` showed **custom metadata** and no encryption.
- `pdftotext` revealed a hint:
  - Archive filename: `KUBGTU_FINANCIAL_DATA_2025.ZIP`
  - Password: `FinanceKubSTU2025!`
  - Claimed checksum: `d9da3d7a3239a1fd4e1a370c2aab8508`

## 2) Carve Embedded ZIP from PDF
`pdfdetach` showed no formal attachments, so I used file carving.

`binwalk` identified a ZIP at offset `0x84A` (decimal `2122`) and an end marker at `0xA75`, so size is `577` bytes.

Extraction command:

```bash
dd if=/media/sf_kali/KUBSTU_Financial_Report_2025.pdf of=embedded.zip bs=1 skip=2122 count=577
```

Validation:

- `md5sum embedded.zip` matched the PDF hint exactly:
  - `d9da3d7a3239a1fd4e1a370c2aab8508`

## 3) Decrypt ZIP
Using the password from page text:

```bash
unzip -P 'FinanceKubSTU2025!' embedded.zip
```

Recovered file:

- `confidential_flag.txt/tmpb1ln4mfr.txt`

It contained:

- `KubSTU{F4k3_Fl4g_D3c3pt10n_Pr0t0c0l_2025_S3cur1ty_Br34ch_Unauthorized_Access_D3t3ct3d}`

This was clearly a trap/decoy (explicit “YOU HAVE BEEN HACKED” bait text + mismatched internal checksum claims).

## 4) Inspect Hidden Metadata Blob
The PDF metadata contains a very large custom field:

- `/HiddenAuditData (...)`

This blob contains hundreds of fake entries (`FAKE`, `DUMMY`, `MOCK`, `DECOY`, `TEST`) and base64-like tokens.

I parsed this field and attempted recursive base64 decoding over candidate tokens. Only one decoded cleanly into a realistic full flag string.

Decoded result:

- `KubSTU{PDF_M3t4d4t4_F0r3ns1cs_4dv4nc3d_Ch4ll3ng3_2025_S3cur3_Emb3dd3d_F1l3_3ncrypt10n_Pr0t0c0l}`

## Final Flag

```text
KubSTU{PDF_M3t4d4t4_F0r3ns1cs_4dv4nc3d_Ch4ll3ng3_2025_S3cur3_Emb3dd3d_F1l3_3ncrypt10n_Pr0t0c0l}
```

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

