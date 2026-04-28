## Challenge Writeup: Quantum Cipher Revenge

The challenge presents a "Quantum Encryption Algorithm" using a 24-qubit system and 31 rounds of operations[cite: 1]. Despite the complex quantum mechanics involving Bloch spheres and unitary gates, the security relies on a flawed implementation that reduces to a simple classical permutation.


1. Analysis of the Vulnerability
The encryption process consists of three main components:
State Initialization: Qubits are rotated into a specific state $|\psi\rangle$ based on parameters $\theta$ and $\phi$[cite: 1].
Key Addition : A unitary gate $U$ is applied based on a secret key. This gate is constructed using a Hamiltonian $H$ that is specifically aligned with the initialization axis[cite: 1]. 
Permutation : A classical shuffle of qubit positions using a fixed `PERMUTATION` array.
Measurement: Before measurement, the initial rotations are reversed.

The Flaw: Because the unitary gate $U$ and the state $|\psi\rangle$ share the same eigenstates, the "encryption" gate only adds a **global phase** to the qubits. [cite_start]In quantum mechanics, global phases are physically unobservable and disappear entirely upon measurement[cite: 1]. This means the secret key and the complex rotations have zero effect on the output. The only operation that actually changes the bits is the **Permutation**.

 2. Exploitation Strategy
Since the quantum gates are irrelevant, the challenge is simply a classical bit-shuffling cipher:
1. The ciphertext is the result of the input bits being permuted 31 times.
2.  The `PERMUTATION` array is provided in plain text by the server.
3.  To recover the flag, we take the ciphertext bits and apply the **inverse** of the permutation 31 times.

 3. Solution Execution
The solve script performs the following:
* Converts the hex ciphertext into a bitstream.
* Divides the bitstream into 24-bit blocks.
* Uses `qiskit` to apply the `PermutationGate(PERMUTATION).inverse()` for 31 iterations per block.
* Converts the resulting bits back into bytes to reveal the flag.

create an short writeup for the same
 
                                                                                                                                                                                                                                            
┌──(venv)─(root㉿kali)-[/home/…/Downloads/practice/theromanx/crypto_quantumcipherrevenge]
└─# python3                                   
Python 3.13.3 (main, Apr 10 2025, 21:38:51) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> #!/usr/bin/env python3
... from qiskit import QuantumCircuit, transpile
... from qiskit_aer import AerSimulator
... from qiskit.circuit.library import PermutationGate
... from Crypto.Util.number import long_to_bytes
... 
... # Hardcoded values extracted from the server
... ct_hex = '103000c976c24513a099764887208a455cca5ad4ba45938089248294e19a9a25fa1beb42ca10329773c0591280d6f89a'
... permutation = [18, 12, 0, 8, 13, 19, 1, 16, 7, 11, 20, 9, 4, 6, 17, 21, 10, 2, 23, 22, 14, 3, 5, 15]
... 
... def decrypt():
...     print("[*] Reconstructing ciphertext bits...")
...     ct_bytes = bytes.fromhex(ct_hex)
...     ct_bits_raw = bin(int.from_bytes(ct_bytes, 'big'))[2:]
...     
...     pad_len = (24 - (len(ct_bits_raw) % 24)) % 24
...     ct_bits = ('0' * pad_len) + ct_bits_raw
... 
...     sampler = AerSimulator(method='statevector', precision='single')
...     inv_perm_gate = PermutationGate(permutation).inverse()
... 
...     pt_bits = ''
... 
...     print("[*] Decrypting blocks...")
...     for i in range(0, len(ct_bits), 24):
...         block = ct_bits[i:i+24]
...         qc = QuantumCircuit(24, 24)
...         
...         for j, b in enumerate(block[::-1]):
...             if b == '1':
...                 qc.x(j)
...                 
...         for _ in range(31):
...             qc.append(inv_perm_gate, list(range(24)))
...             
...         qc.measure(range(24), range(24))
...         
...         tqc = transpile(qc, sampler)
...         job = sampler.run(tqc, shots=1)
...         res = job.result().get_counts()
...         
...         pt_block = list(res.keys())[0]
...         pt_bits += pt_block
... 
...     pt_bytes = long_to_bytes(int(pt_bits, 2))
...     print("\n[+] Decrypted Flag:", pt_bytes.decode(errors='ignore').strip('\x00'))
... 
... if __name__ == '__main__':
...     decrypt()
...     
[*] Reconstructing ciphertext bits...
[*] Decrypting blocks...

[+] Decrypted Flag: TRX{8L0CH_5PH3R3_150M0RPH15M_4ND_GL084L_PH453}
>>> 

Flag: TRX{8L0CH_5PH3R3_150M0RPH15M_4ND_GL084L_PH453}


---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
