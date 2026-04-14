The board is a hardwired 7-bit ASCII comparator bank built in KiCad, not a programmable device. The single input header J1 carries IN0 through IN6, and the 19 LED outputs each light for one specific character. By simulating the TTL logic in smart-brick-v2.kicad_pcb, the lit LED positions map to:

U
M
A
S
S
{
I
n
_
T
h
3
_
G
4
t
3
s
}
So the flag is UMASS{In_Th3_G4t3s}.

I used the KiCad PCB netlist and simulated the 74LS gate network directly, so no hardware was needed.

smart brick v2


* [🔙 Back to Hardware Directory](../)
* [🔙 Back to Hardware Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
