# Challenge Writeup

**Author:** lexu  
**Flag format:** `ping{.*}`

## Initial Recon

The binary is a stripped 64-bit ELF:

```bash
file /media/sf_kali/chall
```

Useful strings:
- `Enter the flag:`
- `Wrong flag!`
- `Correct flag!`

## Static Analysis Summary

- Main logic is at `0x4035cc`.
- Input is read into `0x41c560` via `scanf("%s", ...)`.
- A generated table is stored at `0x41a480`.
- Final comparison routine (called near `0x403544`) compares `0x41a480[i]` with `0x419460[i]`.

Important observation: target values at `0x419460` are interleaved.  
For each index `i`, the checker uses:

- `target[i] = dword(0x419460 + 8*i + 4)`  

And generation formula in the helper functions:

```c
arr[i] = 0xdeadbeef ^ input[i] ^ (i * 0x1337) ^ (prev * 0xabcd);
```

Where:
- `prev = 0x539` for `i = 0`
- `prev = i - 1` for `i > 0`

So recovery is direct:

```c
input[i] = target[i] ^ 0xdeadbeef ^ (i * 0x1337) ^ (prev * 0xabcd);
```

## Recovered Flag

`ping{n0_c0mp1l3r_w45_hur7_dur1ng_m4k1ng_7h15_ch4ll}`

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Reverse Engineering Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
