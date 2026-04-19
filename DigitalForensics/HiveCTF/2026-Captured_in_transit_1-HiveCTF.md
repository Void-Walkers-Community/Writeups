# Captured in Transit I — Writeup

## Challenge

**Name:** Captured in Transit I  
**Category:** Forensics / Network  
**Prompt:**  
> We sent a vibecoded image over the wire real quick for use in our graphics department. Let's hope no one captures it, that would be embarrassing!

The challenge provides a packet capture. The goal is to recover the transmitted image and read the flag from it.

## Given Files

```text
Captured_In_Transit_I.zip
```

After extracting the archive, the important file is a packet capture:

```bash
unzip Captured_In_Transit_I.zip
ls -la
```

Typical output:

```text
captured_in_transit_i.pcap
```

## Initial Analysis

Since the prompt says an image was sent “over the wire,” the first thing to check is whether the packet capture contains a file transfer over a common protocol such as HTTP.

Open the capture in Wireshark:

```bash
wireshark captured_in_transit_i.pcap
```

Then check:

```text
File → Export Objects → HTTP
```

This reveals that an image file was transferred over HTTP.

## Method 1: Extract with Wireshark

In Wireshark:

1. Open the `.pcap`.
2. Go to:

   ```text
   File → Export Objects → HTTP
   ```

3. Look for the transferred PNG file.
4. Save it as:

   ```text
   secret.png
   ```

5. Open the image:

   ```bash
   xdg-open secret.png
   ```

The flag is visible in the recovered image.

## Method 2: Extract with `tshark`

The same result can be achieved from the command line.

First, extract HTTP objects:

```bash
mkdir extracted
tshark -r captured_in_transit_i.pcap --export-objects http,extracted/
```

Then check the extracted files:

```bash
ls -la extracted/
file extracted/*
```

One of the extracted files is a PNG image. Rename it if needed:

```bash
mv extracted/secret.png secret.png
```

Open it:

```bash
xdg-open secret.png
```

## Method 3: Manual Stream Reconstruction

If the file is not immediately obvious, follow the HTTP stream:

```bash
tshark -r captured_in_transit_i.pcap -Y "http" -V
```

Or in Wireshark:

```text
Right click HTTP packet → Follow → TCP Stream
```

The HTTP response contains PNG data. A PNG file starts with this magic header:

```text
89 50 4E 47 0D 0A 1A 0A
```

After exporting the raw response body, the file can be verified:

```bash
file secret.png
```

Expected result:

```text
secret.png: PNG image data
```

## Flag

Opening the extracted image reveals:

```text
HiveCTF{p4ck3t_p34k3r_pr0}
```

## Final Answer

```text
HiveCTF{p4ck3t_p34k3r_pr0}
```

## Key Takeaways

This challenge demonstrates a classic network-forensics workflow:

- Inspect packet captures for transferred files.
- Use Wireshark’s **Export Objects** feature.
- Recognize common file signatures such as PNG magic bytes.
- Reconstruct files from captured network traffic.
