# DLSS — writeup

**Category:** Reverse engineering / machine learning  
**Flag:** `d4rk{hinjaku!hinjaku!}`

## Challenge

The challenge provided a museum simulation and a PyTorch checkpoint, `model.pt`. The description said to collect copies of paintings, use 4xup's decoder, “blow it up 4x,” and never show anyone “the negative.” It also warned that the museum contained forgeries.

## 1. Inspect the museum assets

After extracting `museum_simulation.dist.zip`, the `assets` directory contained eight encrypted paintings, their locked preview images, and a manifest. The encrypted files were named `frame_01.png.enc` through `frame_08.png.enc`. The manifest identified frame 6 as the painting in the gold frame.

The program was a compiled Linux Python application. Extracting printable strings from `museum_obby.bin` revealed a `crypto_utils` module, including the names `_keystream`, `encrypt_bytes`, and `decrypt_bytes`, references to SHA-256 and a nonce, and the embedded passphrase:

```text
unga-bunga-ved-in
```

The game decrypts a painting when the player downloads it. Instead of navigating the platformer, I recovered the encryption format by testing the likely SHA-256 constructions against the known PNG signature and checking that the complete decrypted files opened successfully.

The format was:

- First 16 bytes: nonce.
- Remaining bytes: ciphertext.
- Key: SHA-256 of the passphrase.
- Keystream block `i`: SHA-256 of `key || nonce || i`, where `i` is a four-byte big-endian counter starting at zero.
- Plaintext: ciphertext XOR keystream.

This script recovers all eight PNGs:

```python
from hashlib import sha256
from pathlib import Path

assets = Path('museum_simulation.dist/assets')
outdir = Path('paintings')
outdir.mkdir(exist_ok=True)
key = sha256(b'unga-bunga-ved-in').digest()

for path in sorted(assets.glob('*.png.enc')):
    blob = path.read_bytes()
    nonce, ciphertext = blob[:16], blob[16:]
    stream = b''.join(
        sha256(key + nonce + counter.to_bytes(4, 'big')).digest()
        for counter in range((len(ciphertext) + 31) // 32)
    )
    plaintext = bytes(a ^ b for a, b in zip(ciphertext, stream))
    (outdir / path.name.removesuffix('.enc')).write_bytes(plaintext)
```

Each decrypted file was a valid 128 × 128 PNG.

## 2. Reconstruct the decoder

`model.pt` was a checkpoint dictionary containing `state_dict`, `size` (128), and `base_channels` (16). The state-dictionary names and tensor shapes exposed a U-Net-style architecture: four encoder blocks, a bottleneck, four transpose-convolution upsampling layers with skip connections, four decoder blocks, and a one-channel output head. Each block consisted of two convolution / batch-normalization / ReLU sequences.

I reconstructed that architecture, loaded the weights, and ran each painting through it in evaluation mode. The model accepts an RGB image normalized to `[0, 1]` and outputs a one-channel image. Its output was a negative, so I inverted it; I then enlarged the result fourfold to inspect and scan it. The relevant inference steps were:

```python
image = Image.open(path).convert('RGB')
x = torch.from_numpy(np.array(image).copy())
x = x.permute(2, 0, 1).float().unsqueeze(0) / 255

with torch.no_grad():
    result = model(x)[0, 0].numpy()

# The network's sigmoid output is a negative image.
positive = Image.fromarray((255 - result * 255).astype('uint8'))
positive = positive.resize((512, 512), Image.Resampling.NEAREST)
```

The complete reconstructed network and inference code used for the successful original-size pass are saved in `dlss/native.py`. The painting files are loaded at their original 128 × 128 size; the script saves the inverted output, which can then be enlarged for QR scanning.

## 3. Decode the QR codes

Most outputs did not yield a readable QR code. Painting 5, a portrait of Jotaro, decoded to:

```text
d4rk{yare.yare.daze}
```

That was a forgery. Painting 6, a portrait of Dio in the gold frame, decoded to:

```text
d4rk{hinjaku!hinjaku!}
```

The user confirmed that the second string is the accepted flag. OpenCV's `QRCodeDetector().detectAndDecode()` successfully read both codes after the decoded images were enlarged fourfold with nearest-neighbor interpolation.

---
* [🔙 Back to AI Directory](../AI)
* [🔙 Back to Main Directory](../README.md)
