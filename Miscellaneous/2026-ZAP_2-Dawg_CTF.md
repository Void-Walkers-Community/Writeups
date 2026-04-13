# DawgCTF Misc Challenge Writeup

## Challenge

Identify the model number of the insulator shown in the image.

Flag format:

`DawgCTF{modelnumber}`

## Solution

The object in the image is a **porcelain suspension insulator** used on power lines.

The key hint was:

> Consider everything you've learned so far, and what clues might be inscribed on the insulator to indicate maybe how it functions or what it is rated for, and use those clues to identify what model it is.

That points to using:

- the **insulator type**
- the **rating / ANSI class**
- the **manufacturer model numbering scheme**

## Identification Process

From the shape, this is not a line-post or station-post insulator. It matches a **short-profile suspension disc insulator**.

Once narrowed to the suspension-disc family, the likely NGK-Locke model numbers are in the form:

- `20S840`
- `20S580`
- `30S255`
- `30S257`
- `40S360`

Using cross-reference tables and suspension-insulator catalogs, the correct match for this insulator is:

- **ANSI class:** `52-3`
- **NGK-Locke model:** `20S840`

## Why `20S840`

Reference tables for porcelain suspension insulators map:

- `ANSI 52-3` -> `NGK-Locke 20S840`

Nearby lookalikes include:

- `30S255` -> ANSI 52-5
- `30S257` -> ANSI 52-6

Those were incorrect for this challenge.

## Flag

`DawgCTF{20S840}`

## References

- Newell porcelain cross-reference tables
- NGK/Locke suspension insulator catalog tables
- Border States NGK-Locke suspension product family listing

* [🔙 Back to Miscallaneous Directory](../Miscellaneous)
* [🔙 Back to Miscellaneous Index Directory](../Miscellaneous/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
