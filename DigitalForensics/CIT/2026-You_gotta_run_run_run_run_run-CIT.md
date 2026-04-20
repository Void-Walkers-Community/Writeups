<img width="1193" height="843" alt="Pasted image 20260419223715" src="https://github.com/user-attachments/assets/e3ffd596-73de-4467-a08a-d62ca958030e" /><img width="1193" height="843" alt="Pasted image 20260419223715" src="https://github.com/user-attachments/assets/a81a9410-29ec-456e-9946-e216f3f781b3" />

The challenge file is a `Windows Registry` file. I used `Fred` to open it in Linux.

The black box flashing is definitely the `command prompt` or the `powershell` flashing.
In windows registry, primary location for persistence for running items is
`Software\Microsoft\Windows\CurrentVersion\Run`

We navigate here using `Fred`

<img width="1525" height="190" alt="Pasted image 20260419224017" src="https://github.com/user-attachments/assets/35b09a1f-7006-4603-8f41-44876f3b4a9e" />

OneDrive looks fine but take a look at `AzureTenant`...
It is executing an `.exe` file called `fj3493.exe` which is definitely not normal.

Hence, the name is our flag!

Flag - `CIT{AzureTenant}`
