Flag
gctf{I_h4t3_d3lph1_4nd_n0w_h0pefully_u2}
("I hate Delphi and now hopefully u2" — appropriate, given this is a Free-Pascal/Lazarus binary protected by a custom VM.)

TL;DR of the reversing path
virt.exe is a 64-bit PE compiled with Free Pascal / Lazarus. It is wrapped by a custom virtualization layer that lives in a separate section named .frost0 (CODE | NOREAD), size 0x38e4 — the "Frostgate" VM.

1. Program flow
Driver at 0x100001710 prints the banner, prompts Enter license key:, reads up to 64 bytes of input into the AnsiString at .bss:0x100024010, then calls the validator entry at 0x1000121a0.

The validator entry (0x1000121a0) does three things:

0x100012290 clears the 64-byte buffer at .bss:0x1000260d0 and copies the input into it; the input length (capped at 64) is stored as a 32-bit int at .bss:0x100026110.

Computes an FPC exception-handler pointer as

handler = 0x100012490 ^ format_check(input) ^ anti_debug() ^ hash(input)
format_check returns 0 iff input matches gctf{...} (length ≥ 7, bytes 0..4 = 'g','c','t','f','{', last byte = '}').
anti_debug returns 0 if no debugger, else 0x1000125c0 ^ 0x100012490 = 0x150 (IsDebuggerPresent + CheckRemoteDebuggerPresent).
hash is ((len*len + len) & 1) * 0x9e3779b97f4a7c15 — always 0, since len*(len+1) is always even.
Pushes handler onto the FPC exception-frame stack at .data:0x1000190a0.

After the driver prints Checking license, please wait..., control reaches the registered handler via the FPC exception machinery.

2. Real validator vs. decoy
Because of the XOR construction:

Normal run, gctf{...} input → handler = 0x100012490 = Function 1 (the real validator; 40-byte loop, uses the VM).
Debugged run → handler = 0x1000125c0 = Function 2 (a decoy).
Function 2 (0x1000125c0) checks

for i in 0..38:  result |= array2[i] ^ input[i] ^ ((i*0x2f + 0xa7) & 0xff)
with array2 at .data:0x100019890. Inverting it gives:

wrong{f1n4l1z3r_d3c0y_n0t_th3_r34l_0n3}
— literally wrong{...} / "finalizer decoy not the real one". It is the trap that triggers only when the program is being debugged.

3. Function 1 (the real check)
Function 1 (0x100012490) does:

state0 = [0x100026110] ^ 0xdeadbeef          # = 40 ^ 0xdeadbeef = 0xdeadbec7
result = 0
if [0x100026110] != 0x28: result |= 1         # length must be 40
for i in 0..39:
    result |= array2[i] ^ f(input[i], state, i)
    state = g(state, input[i], i)
success = (result == 0)
array2 is at .data:0x100019860 (40 bytes).

Both f (0x100012340) and g (0x100012410) are tiny stubs that push 0x241; call 0x100029000 — i.e., they are virtualized by the Frostgate VM. The VM dispatcher in .frost0 is heavily self-modifying (each step does pushf; mov DWORD [rip+1], <bytes>; popf; <bytes get overwritten>), so static tracing is impractical.

4. Emulation instead of static analysis
Rather than reversing the VM by hand, I loaded the whole PE image into Unicorn (every section mapped RWX so the SMC works) and called f / g directly via the Windows x64 calling convention:

f(rcx=input_byte, rdx=state, r8=i)  -> byte
g(rcx=state, rdx=input_byte, r8=i)   -> 32-bit new state
A quick probe showed f is a bijection on its first argument for every (state, i) pair (256 unique outputs across 256 inputs). So each input[i] can be recovered independently as

input[i] = f⁻¹( array2[i] ; state, i )
and the state for the next iteration is g(state, input[i], i).

5. Recovering the flag
Iterating from state₀ = 0xdeadbec7:

i= 0  'g' 0x67   i=10  '3' 0x33   i=20  'n' 0x6e   i=30  'e' 0x65
i= 1  'c' 0x63   i=11  '_' 0x5f   i=21  'd' 0x64   i=31  'f' 0x66
i= 2  't' 0x74   i=12  'd' 0x64   i=22  '_' 0x5f   i=32  'u' 0x75
i= 3  'f' 0x66   i=13  '3' 0x33   i=23  'n' 0x6e   i=33  'l' 0x6c
i= 4  '{' 0x7b   i=14  'l' 0x6c   i=24  '0' 0x30   i=34  'l' 0x6c
i= 5  'I' 0x49   i=15  'p' 0x70   i=25  'w' 0x77   i=35  'y' 0x79
i= 6  '_' 0x5f   i=16  'h' 0x68   i=26  '_' 0x5f   i=36  '_' 0x5f
i= 7  'h' 0x68   i=17  '1' 0x31   i=27  'h' 0x68   i=37  'u' 0x75
i= 8  '4' 0x34   i=18  '_' 0x5f   i=28  '0' 0x30   i=38  '2' 0x32
i= 9  't' 0x74   i=19  '4' 0x34   i=29  'p' 0x70   i=39  '}' 0x7d
End-to-end re-emulation of Function 1's loop produces xor = 0x00 at every position, so result = 0 and the program prints:

[+] Access granted. The key you entered is the flag.
Files
flag.txt — the flag, plain text.
writeup.md — this writeup. 

---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
