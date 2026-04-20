# Dog Barking Writeup

## Challenge

**Name:** Dog barking  
**Category:** Forensics / Audio  
**Given file:** `challenge.wav`  
**Flag format:** `CIT{example_flag}`

The challenge gives a `.wav` file that sounds like a sequence of dog barks. Since the barks are repeated in a very regular pattern, the audio is likely encoding data rather than being normal recorded dog noise.

## Initial Analysis

First, inspect the file type:

```bash
file challenge.wav
soxi challenge.wav
```

Output:

```text
challenge.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 16000 Hz

Input File     : 'challenge.wav'
Channels       : 1
Sample Rate    : 16000
Precision      : 16-bit
Duration       : 00:01:17.80
Sample Encoding: 16-bit Signed Integer PCM
```

The audio is mono PCM at 16 kHz. Listening to the file shows that the barks are short and separated by small pauses. More importantly, the barks are not all the same pitch.

## Idea

The barks act like symbols.

After segmenting the audio into individual bark events and checking the dominant frequency of each bark, three useful symbol types appear:

| Symbol type | Approx frequency | Meaning |
|---|---:|---|
| Low bark | around `490 Hz` | bit `0` |
| Higher bark | around `590 Hz` or higher | bit `1` |
| Separator bark | around `530 Hz` | byte separator |

There are `269` bark events total.

This makes sense because:

```text
30 bytes * 8 bits = 240 data barks
29 separators     = 29 separator barks
240 + 29          = 269 total barks
```

So the audio is encoding 30 ASCII bytes, with one separator between each byte.

## Solver

The script below:

1. Loads the WAV file.
2. Splits the audio into bark events using RMS energy.
3. Calculates the dominant frequency of each bark using FFT.
4. Converts each bark into `0`, `1`, or `|`.
5. Splits on `|` to get 8-bit binary chunks.
6. Converts each byte into ASCII.

```python
#!/usr/bin/env python3
import sys
import wave
import numpy as np

if len(sys.argv) != 2:
    print(f"Usage: {sys.argv[0]} challenge.wav")
    sys.exit(1)

path = sys.argv[1]

with wave.open(path, "rb") as w:
    sr = w.getframerate()
    samples = np.frombuffer(w.readframes(w.getnframes()), dtype=np.int16).astype(np.float64)

# 20 ms frames are enough to detect each bark burst.
frame = int(0.02 * sr)
hop = frame

rms = []
for i in range(0, len(samples) - frame, hop):
    chunk = samples[i:i + frame]
    rms.append(np.sqrt(np.mean(chunk * chunk)))

rms = np.array(rms)
mask = rms > 1000

# Convert the RMS mask into bark segments.
segments = []
start = None
for i, active in enumerate(mask):
    if active and start is None:
        start = i
    elif (not active or i == len(mask) - 1) and start is not None:
        end = i if not active else i + 1
        duration = (end - start) * hop / sr
        if duration > 0.05:
            segments.append((start * hop, end * hop))
        start = None

symbols = []
frequencies = []

for a, b in segments:
    x = samples[int(a):int(b)]
    x = x - np.mean(x)

    # FFT with a Hann window to estimate the strongest frequency.
    window = np.hanning(len(x))
    spectrum = np.abs(np.fft.rfft(x * window))
    freqs = np.fft.rfftfreq(len(x), 1 / sr)

    valid = np.where((freqs > 100) & (freqs < 2000))[0]
    dom = freqs[valid[np.argmax(spectrum[valid])]]
    frequencies.append(dom)

    # Separator barks cluster around 530 Hz.
    if 510 <= dom <= 550:
        symbols.append("|")
    # Higher-pitched barks encode bit 1.
    elif dom >= 550:
        symbols.append("1")
    # Lower-pitched barks encode bit 0.
    else:
        symbols.append("0")

stream = "".join(symbols)
chunks = stream.split("|")

print(f"[+] segments: {len(segments)}")
print(f"[+] separators: {stream.count('|')}")
print(f"[+] chunks: {len(chunks)}")
print(f"[+] chunk sizes: {sorted(set(map(len, chunks)))}")

flag = "".join(chr(int(chunk, 2)) for chunk in chunks)
print(f"[+] decoded: {flag}")
```

Run it:

```bash
python3 solve.py challenge.wav
```

Output:

```text
[+] segments: 269
[+] separators: 29
[+] chunks: 30
[+] chunk sizes: [8]
[+] decoded: CIT{b4rking_up_th3_wr0ng_tr33}
```

## Flag

```text
CIT{b4rking_up_th3_wr0ng_tr33}
```

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

