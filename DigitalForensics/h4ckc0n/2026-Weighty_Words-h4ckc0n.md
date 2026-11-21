Weighty Words — Writeup
The challenge gave us a PDF called paper.pdf, along with the hint:
“Old tongues call picture a thousand words, Verily this scroll hath few words, yet it groaneth like a cart of stones”

That made the file size the first thing worth investigating. The PDF contained only five pages of text, but it was roughly 122 MB—far larger than expected for a document with no visible photographs.
Inspecting the PDF
I checked the PDF’s metadata and extracted its text. It was written as an old-fashioned treatise about hiding information, with sections mentioning things such as metadata, image steganography, file carving, and memory forensics.

Most of those sections appeared to be distractions. The PDF had no ordinary embedded images or attachments that explained its size.

I then examined the PDF’s internal objects and found the reason it was so large: an embedded TrueType font contained an unusual table called GSUX, which was approximately 122 MB by itself. A normal embedded font would not need a table anywhere near that size, making it a strong place to look for hidden data.
Recovering the hidden file
The data inside the table initially looked random. Its first few bytes were:
00 3a 7d 5b 5d 03 03 07
I tested simple byte transformations and found that subtracting 3 from every byte, wrapping around when necessary, changed the beginning into:
fd 37 7a 58 5a 00 00 04
That starts with the magic bytes of an XZ-compressed file. This showed that the data had been shifted by 3 before being hidden inside the font table.

After reversing the shift and decompressing the XZ data, I obtained a file of approximately 614 MB. Its header began with SIMPLE, identifying it as a FITS file—a format used for scientific and astronomical image data.

The extraction process can be reproduced with Python:
import lzma
import pymupdf
from fontTools.ttLib import TTFont
from io import BytesIO

# Open the PDF and extract the embedded font stream.
pdf = pymupdf.open("paper.pdf")
font_data = pdf.xref_stream(19)

# Find the offset and length of the suspicious font table.
font = TTFont(BytesIO(font_data), lazy=True)
table = font.reader.tables["GSUX"]

hidden_data = font_data[table.offset : table.offset + table.length]

# Reverse the byte shift.
xz_data = bytes((byte - 3) % 256 for byte in hidden_data)

# Decompress the recovered XZ file.
fits_data = lzma.decompress(xz_data)

with open("recovered.fits", "wb") as output:
    output.write(fits_data)

print("Recovered FITS file:", len(fits_data), "bytes")
This requires PyMuPDF and fonttools.
The decoy
The FITS file contained multiple sections, including a small image named PREVIEW and a much larger image named SCI.

The preview immediately displayed:
d4rk{n1c3_try_but_n0}
That was clearly a decoy: it literally says “nice try but no.” The FITS header also contained a Base64-encoded copy of the same decoy, so finding it twice did not make it the real answer.
Examining the main image
The larger SCI image was 16,512 × 9,288 pixels and stored as floating-point values. Its header described a Bayer pattern. A normal rendering looked mostly like noise, so I examined the individual pixel values instead.

One set of pixels contained a small number of unusually bright values. Isolating those bright pixels revealed faint writing near the lower-right area of the image.

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

