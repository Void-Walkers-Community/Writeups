# The Skeleton Key — Writeup

## Challenge Summary
A Cisco Packet Tracer lab (`3_in_1_1.pkt`) had switch warnings and access-level confusion. We needed to identify the failing interface and recover the hidden flag without wiping config.

## Method
1. Decrypt the `.pkt` file using **Unpacket**:
   - `python3 unpacket.py /media/sf_kali/3_in_1_1.pkt -o /home/parth/HACK/3_in_1_1.xml`
2. Inspect the generated XML for:
   - security/audit logs,
   - interface config lines,
   - embedded secrets/notes.
3. Correlate the event logs with switch configs:
   - Log confirms unauthorized MAC and auto-shutdown on `Fa0/2`.
   - Core config includes hidden descriptive text containing the flag.

## Key Findings
- Log event:
  - `Detected unauthorized MAC on Fa0/2. Port shutdown triggered.`
- Hidden string appears in interface description on `CORE_ROOT`.

## Flag
`KubSTU(gazebo_is_stronger_than_tarask)`

## Remediation (No Config Wipe)
Recommended minimal operational fix on access switch:
- Reset/re-enable affected interface,
- Keep port-security enabled,
- Set violation mode to `restrict` to reduce noisy err-disable behavior,
- Save running config.

Example command flow:
```bash
enable
conf t
interface fa0/2
switchport mode access
switchport access vlan 10
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
shutdown
no shutdown
end
wr mem
```



---
* [🔙 Back to Cloud Security Directory](../)
* [🔙 Back to Cloud Security Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
