Overview

We're given a zip file called s0rry_in_4dv4nc3.zip. The challenge description says most of the contents are redundant or stale, but a few records still reflect the system's original processing format. Focus on what remains consistent.


 Solve

Opening the zip, there are around 10,000 files spread across folders like archive/, logs/, users/, tmp/, and reports/. Each file has a similar structure — a small text file with key-value pairs:

timestamp=178087
profile=gamma
uid=5463
state=disabled
part=01
data=UZMO5D


So every file has a profile, a state, a part number, and a data field.

The first thing I noticed was the profile values — most files are alpha, beta, gamma, or omega, with states like pending, archived, or disabled. With thousands of files per profile this is clearly the noise.

I filtered by state and noticed something off — there were only 7 files with state=active, and all of them had profile=delta. That's the odd one out. The filenames even follow a pattern: sys_XXXXX_01.rec through sys_XXXXX_07.rec.

Sorted by part number and grabbed the data field from each:


VURDVEZ7 
dzNsbF83
aDQ3X3c0
NW4nN183
MF9oNHJk
X3c0NV8x
Nz99

Concatenated: VURDVEZ7dzNsbF83aDQ3X3c0NW4nN183MF9oNHJkX3c0NV8xNz99

base64 decoding dives

UDCTF{w3ll_7h47_w45n'7_70_h4rd_w45_17?}
---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
