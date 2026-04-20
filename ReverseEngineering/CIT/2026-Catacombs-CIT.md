<img width="1189" height="659" alt="Pasted image 20260419190910" src="https://github.com/user-attachments/assets/87466579-108e-4611-9d9d-97cd6318bfee" />

No real hint, just the file.

Type of file:-

```
catacombs: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=70957f34dad6e320b16a90233b16efe644686655, for GNU/Linux 4.4.0, not stripped
```

Since the file is statically linked, it has a lot of noise and is of a bigger size but since it is also `not stripped` we can start directly at the `main` function.

On running the file to check what it does:-

<img width="965" height="521" alt="Pasted image 20260419191128" src="https://github.com/user-attachments/assets/59188302-7d15-4ad8-8269-3b07949d027f" />

We see `ACCESS DENIED` which means if the string is correct, we might get something like `ACCESS GRANTED`

###### Analysis in `ghidra`:-

<img width="516" height="304" alt="Pasted image 20260419191247" src="https://github.com/user-attachments/assets/068abd1a-6777-4f73-96e4-1a6f73dba7e5" />

We found the section of the code which prints `ACCESS DENIED` and `ACCESS GRANTED` in the `main` function itself.
The flag is present in plain sight!

Flag - `CIT{3R2rA2J0PdFH}`


---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


