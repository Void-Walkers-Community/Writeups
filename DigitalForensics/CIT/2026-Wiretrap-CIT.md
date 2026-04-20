# Wiretap Writeup

## Challenge

**Name:** Wiretap  
**Category:** Forensics
**Author:** hypnos

> Ton, they're listening to the phones meet me down at the badabing.

We are given an audio file:

```text
beep_beep_boop.wav
```

The challenge hint strongly suggests old telephone audio. The filename also points toward phone tones and modem-style sounds.

---

## Goal

Recover the hidden flag from the provided WAV file.

Final flag:

```text
CIT{g3t_0ff_th3_ph0n3_1m_0n_th3_1ntern3t}
```

---

## Step 1: Inspect the audio file

First, check the file type and basic metadata.

```bash
file beep_beep_boop.wav
soxi beep_beep_boop.wav
```

This confirms that the challenge file is a normal WAV audio file.

Because the challenge references phones, the next useful step is to listen to it and inspect the waveform/spectrogram.

```bash
ffplay beep_beep_boop.wav
```

The audio contains two interesting parts:

1. Initial telephone keypad tones
2. A modem-like data transmission

That suggests the file contains both **DTMF dialing tones** and **dial-up modem data**.

---

## Step 2: Decode the DTMF tones

DTMF tones are the sounds produced when pressing buttons on a telephone keypad.

A quick way to decode them is with `multimon-ng`:

```bash
multimon-ng -t wav -a DTMF beep_beep_boop.wav
```

The decoded digits are:

```text
4155551998
```

This looks like a phone number. It is probably there to confirm that the audio is phone-related, but the flag is not directly in the number.

---

## Step 3: Identify the modem audio

After the DTMF section, the audio sounds like a classic dial-up modem.

The challenge theme and the age of the fake web request later point toward an old modem standard. A common low-speed modem format used over phone lines is **Bell 103**.

Bell 103 uses audio frequency-shift keying, usually with two separate channels:

- Originate side
- Answer side

The important part is that the modem audio can be demodulated back into serial data.

---

## Step 4: Demodulate the Bell 103 data

There are several tools that can decode Bell 103-style modem audio. Depending on the local setup, possible options include:

- `minimodem`
- GNU Radio
- custom Python FSK demodulation
- audio modem decoder scripts

Using `minimodem`, try Bell 103-compatible decoding at 300 baud:

```bash
minimodem --rx 300 -f beep_beep_boop.wav
```

If the output looks reversed, noisy, or empty, try the opposite mark/space channel options or invert the signal depending on the tool version.

A useful workflow is to trim the modem section first, then decode only that part:

```bash
sox beep_beep_boop.wav modem.wav trim 10
minimodem --rx 300 -f modem.wav
```

The recovered data is an old HTTP request and response.

---

## Step 5: Recover the HTTP request

The decoded modem data contains a request similar to:

```http
GET /SiliconValley/Heights/4721/diary.html HTTP/1.0
Host: www.geocities.com
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 95)
```

This is a fake retro web request to a GeoCities-style page.

The response contains HTML, including small embedded SVG/image-like content. The flag is not simply printed as plain text; it is hidden visually in the returned page data.

---

## Step 6: Extract and inspect the HTML/SVG data

Save the decoded modem output into a file:

```bash
minimodem --rx 300 -f modem.wav > decoded.txt
```

Search for HTML or SVG content:

```bash
grep -iE "html|svg|CIT|flag" decoded.txt
```

The page contains tiny SVG scanline-style fragments. Extract those SVG fragments and render or inspect them visually.

For example, if the SVG content is saved into files:

```bash
convert svgline0.svg svgline0.png
convert svgline1.svg svgline1.png
convert svgline2.svg svgline2.png
```

Or open them directly in a browser/image viewer.

The small rendered scanlines reveal the flag text.

---

## Step 7: Read the flag

The rendered SVG/scanline output spells:

```text
CIT{g3t_0ff_th3_ph0n3_1m_0n_th3_1ntern3t}
```

---

## Flag

```text
CIT{g3t_0ff_th3_ph0n3_1m_0n_th3_1ntern3t}
```
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
