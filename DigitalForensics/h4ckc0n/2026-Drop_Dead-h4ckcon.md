Dead Drop
The challenge gave us dead_drop.zip and this hint:
“I have heard of a foreign six, travelers call it 6^2”

The archive contained one file, dead_drop.jpg. The hint 6² evaluates to 36, which turned out to be both the Steghide passphrase and a clue for decoding the next layer.
1. Extract the hidden file
The JPEG looked like an ordinary photograph of palm trees. Running Steghide with 36 as the passphrase recovered a hidden text file:
steghide extract -sf dead_drop.jpg -p 36 -xf payload.txt
The extracted file contained a long string made up of uppercase letters and digits. That character set, together with the hint, suggested base 36.
2. Decode the base-36 data
The entire string represented one large base-36 integer. Converting it to bytes produced data beginning with the XZ archive signature:
from pathlib import Path
import lzma
import sys

sys.set_int_max_str_digits(0)

encoded = Path("payload.txt").read_text().strip()
number = int(encoded, 36)
compressed = number.to_bytes((number.bit_length() + 7) // 8, "big")

disk_image = lzma.decompress(compressed)
Path("filesystem.img").write_bytes(disk_image)
Decompression produced a 32 MB ext filesystem image.
3. Investigate the filesystem
The filesystem contained a home directory, a Kerberos keytab, a research file, fonts and some deleted files. The user’s shell history was especially useful:
cat bracket.gcode | head -n 20
klist -k /etc/krb5.keytab
sudo cp /etc/krb5.keytab /var/backups/backup.keytab
...
rm bracket.gcode
cd ~/Music
rm tune.mid
This pointed to two deleted files worth recovering: bracket.gcode and tune.mid.

The file /home/agent/README also contained a SHA-256 hash for the final URL. That gave us a way to distinguish the real fragments from decoys.
4. Recover the URL fragments
The final URL was split into five numbered pieces, each hidden using a different technique.
Fragment 1 — Kerberos keytab and Brainfuck
The key material in /etc/krb5.keytab contained pieces of a Brainfuck program. Sorting the keytab entries by their key version number and joining their key data reconstructed the program.

Running it produced:
1:https://hljuqfqgsdycotnhdndt
There was another encoded program in /var/backups/backup.keytab, but its output led to a different host. The final URL hash showed that it was a decoy.
Fragment 2 — Deleted MIDI and Whitespace
The deleted tune.mid was recoverable from inode 20. Its note-on timing values contained a program written in the Whitespace esoteric language.

For each note-on event, indexed from zero, I calculated:
value = (event.time - index) % 8 - 1
The resulting values alternated between two ranges. Normalizing them produced a stream where 0, 1 and 2 represented Whitespace’s space, tab and newline characters.

The recovered program used push and character-output instructions and printed:
2:.supabase.co
Fragment 3 — Font file and Befunge
The file /usr/share/fonts/misc/vga8.fon contained a short Befunge program after the VGA8 marker. The program stored its message backwards inside a quoted string and printed it in reverse order.

Its output was:
3:/storage/v1/object/public/
Another font, term.fon, contained a Deadfish program that produced a similar /sign/ path. The SHA-256 check later showed that /public/ was the correct fragment.
Fragment 4 — Research file and Piet
The research file session_0427.nwb was an HDF5 file. Inside it, the dataset:
acquisition/calcium_frame/data
contained a single-row Piet program represented by coloured blocks. Each codel was eight pixels wide.

Reading the colour transitions as Piet instructions produced:
4:image/
Fragment 5 — Deleted G-code
The deleted bracket.gcode was recoverable from inode 26. Its extrusion values had three decimal digits, for example:
E1.096
E2.046
Those decimal portions represented ASCII values. After decoding them, every third character formed a reversed message. Reversing it gave:
5:Screenshot%20from%202026-11-09%2017-32-54.png
The remaining G-code file, old_bracket.gcode, contained a similar filename ending in 45.png; that was another decoy.
5. Assemble and verify the URL
Removing the numeric prefixes and joining the five fragments gave:
https://hljuqfqgsdycotnhdndt.supabase.co/storage/v1/object/public/image/Screenshot%20from%202026-11-09%2017-32-54.png
Its SHA-256 hash was:
9f2bce0edb56e82e2e5ecf5e9585f9fb19495c6767f1dd1920742986119ca0f8
That exactly matched the hash in the filesystem’s README, confirming the correct combination of fragments. The URL led to the flag image. 

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
