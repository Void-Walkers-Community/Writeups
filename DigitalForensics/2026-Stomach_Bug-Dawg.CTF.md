# Stomach Bug Writeup

## Challenge Overview

The challenge description said:

> “You ever had a server break so badly it vomits all over your network? It just keeps spewing forever...”

That strongly suggested the problem was not a normal web page challenge, but a malformed or intentionally noisy HTTP response. The goal was to extract the meaningful data from the endless stream.

---

## Enumeration

I started by requesting the site with `curl` instead of opening it in a browser:

```bash
curl -skN --http1.1 -D headers.txt https://stomachbug.umbccd.net/ --max-time 5 -o body.bin
cat headers.txt
head -c 2048 body.bin
xxd -g1 -l 512 body.bin
```

The response headers were interesting:

```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Transfer-Encoding: chunked
content-disposition: attachment; filename="spew.txt"
```

So the server was not returning HTML. It was sending a streamed binary attachment called `spew.txt`.

Looking at the body showed a repeated pattern. The output alternated between:

1. printable junk lines, such as:

```text
!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefgh
```

2. structured numbered lines, such as:

```text
|000|89504e470d0a1a0a0000000d494844520000027100000271080000000008e6bbe40
|001|00014fa4944415478daeddac192db38804441ffff4fcf1ef7b266d703e48d682a71
```

The first numbered line immediately stood out because `89504e470d0a1a0a` is the standard PNG file signature.

That meant the useful payload was almost certainly hidden in those `|NNN|...` lines.

---

## Understanding the Stream

To inspect the raw chunked transfer encoding, I used:

```bash
curl -skN --http1.1 --raw -D - https://stomachbug.umbccd.net/ --max-time 5 | sed -n '1,120p'
```

That showed the server was sending repeated HTTP chunks of size `49` hex, with the junk line and the numbered line alternating inside the stream.

At first glance, it looked like I could simply extract each numbered line, convert the hex to bytes, and rebuild the file. But there was a problem: each numbered line contained **67 hex characters**, which is an odd number.

Hex decoding requires pairs of characters per byte, so each line by itself was invalid. That meant the data was being split across line boundaries at the nibble level, not the byte level.

So the correct approach was:

- ignore the printable junk lines
- keep only lines matching `|NNN|<hex>`
- concatenate all hex fragments together
- stop once the PNG end chunk appears
- decode the full hex stream into a file

---

## Reconstructing the PNG

I wrote the following Python script:

```python
import re, sys

hexbuf = ""
expected = 0
end_chunk = "0000000049454e44ae426082"   # PNG IEND chunk

for line in sys.stdin:
    line = line.rstrip('\n')
    m = re.fullmatch(r'\|(\d{3})\|([0-9a-fA-F]+)', line)
    if not m:
        continue

    idx = int(m.group(1))
    hx = m.group(2)

    if idx != expected:
        print(f'[!] numbering jump: got {idx:03d}, expected {expected:03d}', file=sys.stderr)
        expected = idx

    hexbuf += hx
    expected += 1

    pos = hexbuf.find(end_chunk)
    if pos != -1:
        hexdata = hexbuf[:pos + len(end_chunk)]
        data = bytes.fromhex(hexdata)

        with open('vomit.png', 'wb') as f:
            f.write(data)

        print(f'[+] wrote vomit.png ({len(data)} bytes, {expected} numbered lines)')
        break
```

Then I piped the server output into the script:

```bash
curl -skN --http1.1 https://stomachbug.umbccd.net/ | python3 recover_png.py
```

This successfully wrote a file called `vomit.png`.

To verify it, I checked the file type:

```bash
file vomit.png
```

It was a valid PNG image.

---

## Analyzing the Image

Opening `vomit.png` revealed a large QR code.

Scanning that QR did not directly produce the flag. Instead, it led to another PNG image. That second image contained a smaller QR code, and scanning that one produced a base64-encoded string.

After base64 decoding, the final flag was revealed:

```text
DawgCTF{1_BL4M3_TH0S3_H4ZM4T_TR5CK3R5}
```

---

## Final Flag

```text
DawgCTF{1_BL4M3_TH0S3_H4ZM4T_TR5CK3R5}
```

---

## Takeaways

This challenge was mainly about recognizing signal inside noise.

The important observations were:

- the response was an endless chunked stream, not a normal webpage
- the printable lines were decoys
- the numbered lines contained the real payload
- each line had an odd number of hex characters, so the hex had to be concatenated before decoding
- the recovered file was a PNG containing nested QR-based stages until the flag was reached

It was a nice challenge because the transport format itself was the puzzle, rather than traditional exploitation.
