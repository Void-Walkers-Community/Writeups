Cool Car Writeup-- i started by checking the image for anything obvious but nothing useful showed up at first glance so i looked at the individual channels instead

the hidden data was stored in the alpha channel specifically in the least significant bit of each alpha value when i extracted that bit plane it revealed a black and white text layer inside the image

that hidden text contained a base64 string

Q0lUezRWdTF1MXpofQ==

after decoding it i got the flag

CIT{4Vu1u1zh}

python code i used

from PIL import Image
import numpy as np

img = Image.open("cool_car.png").convert("RGBA")
arr = np.array(img)

alpha_lsb = (arr[:, :, 3] & 1) * 255
Image.fromarray(alpha_lsb.astype("uint8")).save("alpha_lsb.png")
print("saved: alpha_lsb.png")

---
* [🔙 Back to Steganography Directory](../)
* [🔙 Back to Steganography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
