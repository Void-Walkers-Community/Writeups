# CTF Writeup: Name Calling

**Challenge Type:** Network Forensics  
**Flag:** `UDCTF{wh4ts_wr0ng_mcf1y}`  
**Difficulty:** Easy / Medium

---

## What is This Challenge?

We have a file called `yousaidwhat.pcapng`. This is a **network capture file**. It records all network traffic. We need to find a hidden flag inside it.

---

## Step 1: Open the File and Look at HTTP Traffic

First, we use **Wireshark** or **tshark** to see what is inside the file.

```bash
tshark -r yousaidwhat.pcapng -Y "http" -T fields -e http.request.uri
```

We can see these HTTP requests:

| File | Result |
|------|--------|
| decoy1.txt | 404 - Not found (fake) |
| decoy2.txt | 404 - Not found (fake) |
| whoareyoucalling.zip | ✅ Found |
| whossliming.jpg | ✅ Found |
| stinky.jpeg | ✅ Found |
| chicken.jpg | ✅ Found |

The decoy files are **traps**. The real files are the ZIP and the images.

---

## Step 2: Extract the Files

We export all HTTP objects from the PCAP file.

```bash
tshark -r yousaidwhat.pcapng --export-objects http,./extracted_files
```

Or in Wireshark: **File → Export Objects → HTTP**

Now we have all files saved to the `extracted_files` folder.

---

## Step 3: Try to Open the ZIP File

We try to open `whoareyoucalling.zip`. But it asks for a **password**.

```bash
unzip whoareyoucalling.zip
# Archive: whoareyoucalling.zip
# [whoareyoucalling.zip] whoareyoucalling.txt password:
```

We need to find the password. Let's look at the images.

---

## Step 4: Check EXIF Data in Images

Images have **metadata** called EXIF. Sometimes secret information is hidden there. We use `exiftool` to read it.

```bash
exiftool extracted_files/*.jpg extracted_files/*.jpeg
```

We look at the output carefully. In `chicken.jpg`, we find something strange in the **Copyright** field:

```
Copyright: 6e6f626f64792063616c6c73206d6520636869636b656e21
```

This looks like **hex code**!

---

## Step 5: Decode the Hex

We convert the hex to text using Python:

```python
bytes.fromhex("6e6f626f64792063616c6c73206d6520636869636b656e21").decode()
```

Output:

```
nobody calls me chicken!
```

This is our **ZIP password**! 🐔

---

## Step 6: Open the ZIP File

```bash
unzip -P "nobody calls me chicken!" whoareyoucalling.zip
```

We extract `whoareyoucalling.txt` and read it:

```bash
cat whoareyoucalling.txt
```

**Flag found:**

```
UDCTF{wh4ts_wr0ng_mcf1y}
```

---

## The Reference 🎬

"Nobody calls me chicken!" is a famous line from the movie **Back to the Future**. The character Biff says this to Marty McFly. The flag `wh4ts_wr0ng_mcf1y` means "What's wrong, McFly?" — another line from the same movie

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
