lets avoid doing math = DawgCTF[0.95,0.05,0.05,0.85,0.1,0.25,0.85,0.1125,0.225]

# Threat Depth Analysis Writeup

## Summary

The log file `/media/sf_kali/threat_depth_analysis.log` contains 120 malware samples.
Each sample includes:

- `Known Threat Depth: <class>`
- `Detected Threat Depth: <class>`

The three classes are:

- `minor`
- `medium`
- `major`

The prompt asks for, for each class in growing order of importance (`minor`, `medium`, `major`):

- accuracy
- false positive rate
- false negative rate

All values must be comma-separated and formatted with:

- exactly one leading zero for decimal values less than 1
- no trailing zeroes

## Extraction

The useful fields can be pulled out with regular expressions:

```regex
Known Threat Depth: (\w+)
Detected Threat Depth: (\w+)
```

That gives 120 ground-truth labels and 120 detected labels.

## Method

Treat each class as a one-vs-all classifier.

For a class `c`:

- `TP`: known = `c` and detected = `c`
- `TN`: known != `c` and detected != `c`
- `FP`: known != `c` and detected = `c`
- `FN`: known = `c` and detected != `c`

Formulas:

```text
accuracy = (TP + TN) / total
false positive rate = FP / (FP + TN)
false negative rate = FN / (FN + TP)
```

## Counts

Each class appears 40 times in the ground truth.

### minor

- `TP = 38`
- `TN = 76`
- `FP = 4`
- `FN = 2`

Metrics:

- accuracy = `0.95`
- false positive rate = `0.05`
- false negative rate = `0.05`

### medium

- `TP = 30`
- `TN = 72`
- `FP = 8`
- `FN = 10`

Metrics:

- accuracy = `0.85`
- false positive rate = `0.1`
- false negative rate = `0.25`

### major

- `TP = 31`
- `TN = 71`
- `FP = 9`
- `FN = 9`

Metrics:

- accuracy = `0.85`
- false positive rate = `0.1125`
- false negative rate = `0.225`

## Final Flag

```text
DawgCTF[0.95,0.05,0.05,0.85,0.1,0.25,0.85,0.1125,0.225]
```

* [🔙 Back to Digital Forensics Directory](../DigitalForensics)
* [🔙 Back to Digital Forensics Index Directory](../DigitalForensics/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
