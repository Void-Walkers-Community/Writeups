Writeup for Shades of Trust
In metadata we found this particular info
Charset:  .:-=+#%@

Finding the hidden pattern
Each pixel has red, green and blue colour values. I inspected their bit planes: images made by displaying a selected bit from each colour value.

The useful pattern appeared in the two least significant bits of the red and green channels. These bits represent small changes in colour, so the pattern was difficult to see in the original image.

To make it visible, I checked each pixel:
If either the red or green channel had a nonzero value in its two least significant bits, I made the pixel white.
Otherwise, I made it black.
In code, value & 3 keeps just those two lowest bits. Combining the red and green results revealed block lettering across the centre of the image.
```
from PIL import Image
import numpy as np

Load the image as an array of RGB pixel values.
image = np.array(Image.open("chall.png"))

Keep pixels where either the red or green channel has
a nonzero value in its two least significant bits.
hidden = (
    ((image[:, :, 0] & 3) != 0) |
    ((image[:, :, 1] & 3) != 0)
)

Convert the result to a black-and-white image.
result = Image.fromarray((hidden 255).astype("uint8"))

Crop around the lettering and enlarge it without smoothing.
result = result.crop((0, 85, 750, 160))
result = result.resize((2250, 225), Image.Resampling.NEAREST)
result.save("extracted-text.png")```

g4un{dvf11_o0y3} was recovered
After using caesar cipher I got d4rk{asc11_l0v3}

