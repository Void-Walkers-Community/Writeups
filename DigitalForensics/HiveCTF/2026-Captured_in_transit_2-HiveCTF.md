# Captured in Transit II — Writeup

## Challenge

The challenge gives us a network capture and says that the image was encrypted this time, so simply intercepting the transfer is not enough.

The important hint is:

> "we made the key something super easy for us to remember"

Since this is the second version of the challenge, the key is likely related to information from the first one.

## Files

```text
Captured_In_Transit_II.zip
```

After extracting:

```bash
unzip Captured_In_Transit_II.zip
ls -la
```

We get a packet capture file.

## 1. Open the capture

Open the capture in Wireshark:

```bash
wireshark capture.pcap
```

Then check the protocol hierarchy or apply an FTP filter:

```text
ftp || ftp-data
```

The capture contains an FTP login and file transfer.

## 2. Recover FTP credentials

Following the FTP control stream reveals the login credentials:

```text
USER nyanstein
PASS s3cr3t_ny4n_p4ss
```

So the FTP credentials are:

```text
username: nyanstein
password: s3cr3t_ny4n_p4ss
```

## 3. Extract the transferred file

The FTP transfer contains a file named:

```text
secret2.png
```

However, exporting it directly does not give a valid PNG. The file is encrypted.

A normal PNG should begin with this magic header:

```text
89 50 4E 47 0D 0A 1A 0A
```

If the extracted file does not start with that header, it means the transferred bytes were altered or encrypted.

## 4. Guess the encryption key

The challenge text says the key is easy for the creators to remember.

From the FTP credentials and challenge theme, the repeated word `NYAN` stands out.

So we try XOR decryption with the repeating key:

```text
NYAN
```

## 5. Decrypt the file

Use this Python script:

```python
from pathlib import Path

enc = Path("secret2.png").read_bytes()
key = b"NYAN"

dec = bytes(b ^ key[i % len(key)] for i, b in enumerate(enc))

Path("secret2_decrypted.png").write_bytes(dec)

print(dec[:8])
```

Run it:

```bash
python3 decrypt.py
```

The output should show the PNG header:

```text
b'\x89PNG\r\n\x1a\n'
```

That confirms the key is correct.

## 6. Open the decrypted image

Open the recovered image:

```bash
xdg-open secret2_decrypted.png
```

The image contains the flag.

## Flag

```text
HiveCTF{y0u_f0und_th3_s3cr3t_k3y_fr0m_p4rt1}
```

## Final Solve Script

```python
from pathlib import Path

enc_path = Path("secret2.png")
out_path = Path("secret2_decrypted.png")

data = enc_path.read_bytes()
key = b"NYAN"

decrypted = bytes(
    byte ^ key[i % len(key)]
    for i, byte in enumerate(data)
)

out_path.write_bytes(decrypted)

if decrypted.startswith(b"\x89PNG\r\n\x1a\n"):
    print("[+] Valid PNG recovered")
    print(f"[+] Wrote {out_path}")
else:
    print("[-] Decryption did not produce a PNG")
    print("[*] First 16 bytes:", decrypted[:16].hex())
```

## Summary

The capture contains an FTP transfer of an encrypted image. The FTP credentials reveal a strong hint toward the key theme. Since the challenge says the key is easy to remember, the repeated XOR key `NYAN` is used to decrypt the transferred file. After XOR decryption, the output becomes a valid PNG containing the flag.

```text
HiveCTF{y0u_f0und_th3_s3cr3t_k3y_fr0m_p4rt1}
```

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
