# 🛠️ Reverse Engineering: Binary Deconstruction

Welcome to the Void-Walkers Assembly Lab. This directory contains our deep dives into compiled binaries, obfuscated scripts, and proprietary protocols. We bridge the gap between machine code and human logic.

---

## 🏗️ Operation Index (Completed Analysis)

| Date | Event / Challenge | Architecture | Writeup Link |
| :--- | :--- | :--- | :--- |
| 2026-03 | **0xFUN CTF** | x86_64 ELF | [View Analysis](./0xFUN-2026/License-Check.md) |
| 2026-02 | **CPython Internals** | Python Bytecode | [View Analysis](./Internals/Infinite-Digits.md) |
| 2026-02 | **Cryptographic LCG** | Algorithm Logic | [View Analysis](./Algos/LCG-Break.md) |
| 2025-11 | **Malware Recon** | Windows PE (x32) | [View Analysis](./Malware/Dropper-01.md) |

---

## 📜 Reversing Standards

When submitting a REV writeup, please ensure the following:
1.  **Entry Point Identification:** Clearly state the `main` or `start` address.
2.  **Logic Mapping:** Include pseudocode or flowcharts for complex algorithms.
3.  **The "Ah-ha!" Moment:** Explain exactly where the vulnerability or hidden flag was located (e.g., a hardcoded key or a bypassed `JNE` instruction).
4.  **Patched Binaries:** If the solution required patching a binary, include the `.patch` file or the modified hex addresses.

> "Code is just a suggestion. Reality is in the assembly." 🌌

---
[🔙 Back to Main Directory](../README.md)
