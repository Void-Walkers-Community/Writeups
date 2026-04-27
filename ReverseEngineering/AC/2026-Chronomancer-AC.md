# Chronomancer

Recovered key:

`CTFAC{13587d487718681377b6b43f539311e990edceaa796d23c8a4f12b8dbd72f105}`

## Summary

The binary is a stripped 64-bit Rust PIE that reads one line from stdin, validates that the line is valid UTF-8, trims trailing `\r` / `\n`, and then feeds the resulting 71-byte buffer into a custom VM-like verifier.

Two user-facing failure paths are important:

- `temporal noise detected`: the frontend text gate failed
- `causal checksum rejected`: the VM ran, but the key failed

The success path prints:

- `timeline stabilized`

## Fast Recon

The landing page only served one file:

- `/chronomancer`

Basic triage showed:

- stripped ELF64 PIE
- Rust runtime strings
- three challenge strings in `.rodata`:
  - `chronokey>`
  - `causal checksum rejected`
  - `timeline stabilized`
  - `temporal noise detected`

The useful static table lives in `.data.rel.ro` around `0x4cbb0` and points to those messages.

## Frontend Gate

The function around `0x9930` is a UTF-8 validator. The caller rejects the input with `temporal noise detected` unless:

1. a full line was read
2. the line is valid UTF-8

After that, the code strips trailing CR/LF by walking backward over decoded code points, then hands the remaining bytes to the real checker.

That distinction matters: many raw byte sequences satisfy parts of the VM, but invalid UTF-8 never reaches the final verifier.

## VM Structure

The challenge logic starts at `0x75c9` and enters a fixed loop at `0x7707`.

Useful observations:

- the first opcode enforces `len(input) == 71`
- the loop body follows a fixed path
- exactly one input byte is consumed per round
- there are 71 rounds
- after the 71st round, the code reaches the final compare at `0x7c42`

The final success condition is:

- error mask at `[rsp+0x320]` must be `0`
- accumulator xor at `r14` must equal `0x20220555a65f8ac4`

## Solve Strategy

Instead of reimplementing the VM by hand, the shortest reliable route is to emulate the native machine code directly and solve each round against the real state.

The loop is ideal for this:

- the path is fixed
- one input byte affects one round
- the per-round check is observable at `0x7acd`, where `AL` holds the byte-check result after XOR
- the correct byte is the unique value that makes `AL == 0`

So the solver:

1. initializes Unicorn with the binary image, stack frame, program blob, and a 71-byte input buffer
2. runs to the first round start at `0x77de`
3. snapshots the emulator state
4. for each byte position:
   - restores the snapshot
   - tries all `0x00..0xff`
   - runs to `0x7acd`
   - keeps the unique byte that makes `AL == 0`
5. advances the real state to the next round boundary
6. after all 71 rounds, verifies the final machine state at `0x7c23`

This yields the printable UTF-8 key:

`CTFAC{13587d487718681377b6b43f539311e990edceaa796d23c8a4f12b8dbd72f105}`

## Verification

Running the original binary with that line produces:

```text
chronokey> timeline stabilized
```

The solver used for the recovery is saved as:

- `ac26/solve_chronomancer.py`

It expects `unicorn` to be installed. A local virtual environment was sufficient:

```bash
python3 -m venv /home/priyanshu/ac26/.venv
/home/priyanshu/ac26/.venv/bin/pip install unicorn
/home/priyanshu/ac26/.venv/bin/python /home/priyanshu/ac26/solve_chronomancer.py
```

---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
