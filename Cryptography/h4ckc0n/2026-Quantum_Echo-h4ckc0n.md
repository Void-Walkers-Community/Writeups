Write-up- Quantum Echo
The transcript exposes which samples were sifted: samples 0–15 have matching Z bases and are tapped into the register seed. The audit also leaks the first seed bit, 1. The controller calibration probes reveal the register operations: the fold rotations are 5 and 11 bits, and the walk rotation is 7 bits.

Using those operations with the 160 masked register readings gives the fold and walk masks s=1d0cc456 and t=ccc78639. The sealing record’s compatibility vectors identify the keystream rule: hash the ASCII material string followed by a 4-byte big-endian counter, starting at zero.

The remaining seal material includes a 16-bit q. Searching its 65,536 possible values against the known d4rk{ flag prefix gives q=1010010010100001; its first bit agrees with the transcript’s leaked bit. With e0=0, the material string is:

1d0cc456:ccc78639:0:1010010010100001


For each counter, compute SHA256(material_ascii || counter_u32_be) and XOR the resulting 32-byte blocks with sealed_flag. The supplied compatibility vectors all match this procedure, and the plaintext is:

d4rk{quantum_echo_reuse_breaks_the_pad}

