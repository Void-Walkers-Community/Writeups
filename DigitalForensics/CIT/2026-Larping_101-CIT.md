# Larping 101 Writeup

## Challenge Overview

We were given a PowerPoint file named `challenge.pptx`. Opening the presentation normally only showed a short joke slide deck about "larping", but no visible flag was present on the slides.

Because `.pptx` files are actually ZIP archives containing XML files and media resources, the next step was to inspect the internal file structure instead of only looking at the visible slides.

## Tools Used

- `file`
- `unzip`
- `grep`
- `strings`

## Initial File Check

First, confirm the file type:

```bash
file challenge.pptx
```

Output:

```text
challenge.pptx: Microsoft PowerPoint 2007+
```

This confirms the file is a modern PowerPoint document, which means it can be extracted like a ZIP archive.

## Extracting the PPTX

Extract the PowerPoint contents:

```bash
mkdir extracted
unzip -q challenge.pptx -d extracted
```

After extraction, the internal structure looked like this:

```text
extracted/
├── [Content_Types].xml
├── _rels/
├── docProps/
└── ppt/
    ├── media/
    ├── presentation.xml
    ├── slideLayouts/
    ├── slideMasters/
    ├── slides/
    └── theme/
```

The visible slide text is normally stored in `ppt/slides/slide*.xml`, but challenge authors can also hide data in other XML files inside the PowerPoint package.

## Searching for the Flag

A recursive search was used to look for the flag format:

```bash
grep -R "CIT{" -n extracted
```

This revealed a match inside an unusual XML file:

```text
extracted/ppt/slides/transitions.xml
```

Reading the file:

```bash
cat extracted/ppt/slides/transitions.xml
```

Relevant section:

```xml
<p:debug>
    <p:log level="info">transition engine initialized</p:log>
    <p:log level="warning">compatibility mode enabled</p:log>

    <p:reserved>
        CIT{l4rp_l4rp_l4rp_s4hur}
    </p:reserved>
</p:debug>
```

The flag was hidden in a fake/debug transition XML section, not in the visible slide content.

## Flag

```text
CIT{l4rp_l4rp_l4rp_s4hur}
```

## Takeaway

PowerPoint files are ZIP-based Office documents. If the flag is not visible in the slides, extract the `.pptx` and inspect the internal XML files, relationships, metadata, and media files. Recursive searching with `grep` is often enough to catch hidden strings in these challenges.

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
