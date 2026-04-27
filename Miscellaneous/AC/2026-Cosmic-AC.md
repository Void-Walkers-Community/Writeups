# Cosmic

Target: `http://c79ec57a8e72a87d.ctf.ac.upt.ro`

Flag: `CTFAC{21789fb2dab01d067f1df6174f184da6a66df2fd54f7c8ce7b98e8a4d0280f89}`

## Summary

The page is only a frontend. The real challenge is `/captcha`, which returns a large JSON object with `995` base64 JPEGs.

Each JPEG is a rendered token made from per-character glyph tiles with randomized noise/artifacts. A single capture is hard to read, but the underlying token stream is fixed across refreshes.

## Solve path

1. Fetched `/captcha` several times and saved the responses.
2. Verified that token widths stay identical across refreshes, so the underlying token at each index is stable.
3. Averaged the same token position across `6` captures to suppress the visual noise.
4. Segmented each token into fixed-width character tiles.
5. Clustered the averaged character tiles to get a noisy substitution alphabet.
6. Ran a simple English n-gram hillclimber over the recovered token stream.
7. The decoded text contained a mid-log flag sequence written as words:

`ctfac leftcurlybrace two one seven eight nine fb two dab hero one d hero sit seven f one df sit one seven four f one eight four da sit a sit sit df two fd five four f seven c eight ce seven b nine eight e eight a four d hero two eight hero f eight nine rightcurlybrace`

Where:

- `hero -> 0`
- `sit -> 6`
- `leftcurlybrace -> {`
- `rightcurlybrace -> }`

That gives:

`CTFAC{21789fb2dab01d067f1df6174f184da6a66df2fd54f7c8ce7b98e8a4d0280f89}`

## Validation

Submitting that exact string to `/check` returns `{"result":"Correct"}`.

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
