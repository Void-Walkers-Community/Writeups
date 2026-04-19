## CTF Writeup: ENigma

### Challenge Overview
The challenge involves an RSA-based encryption server that allows users to select their own public exponent $e$. The objective is to force the "encryption" process to return the original plaintext, thereby triggering the server to release the flag.

* **Server:** `enigma.aws.jerseyctf.com:9001`
* **Goal:** Choose an exponent $e$ such that $m^e \equiv m \pmod n$.

---

### Initial Analysis
The server provides the value of the totient $\phi(n)$ directly and implements three specific sanity checks to prevent the use of a "useless" public key:

1.  $e \neq 1$
2.  $e \pmod{\phi(n)} \neq 0$
3.  $\gcd(e, \phi(n)) = 1$

While these checks are designed to ensure a valid RSA configuration, providing the totient $\phi(n)$ to the user creates a massive cryptographic vulnerability.

### Key Insight
The solution lies in **Euler's Theorem**, which states that for any integer $m$ coprime to $n$:

$$m^{\phi(n)} \equiv 1 \pmod n$$

By multiplying both sides by $m$, we can derive the identity function:

$$m^{\phi(n) + 1} \equiv m \pmod n$$

By setting our public exponent **$e = \phi(n) + 1$**, the "encryption" becomes an identity transformation where the ciphertext is identical to the plaintext.

### Verification of Sanity Checks
Choosing $e = \phi(n) + 1$ successfully bypasses all server restrictions:
* **Check 1:** Since $\phi(n)$ is a large positive integer, $\phi(n) + 1 \neq 1$.
* **Check 2:** $(\phi(n) + 1) \pmod{\phi(n)} = 1$, which is not $0$.
* **Check 3:** $\gcd(\phi(n) + 1, \phi(n))$ is always $1$ (consecutive integers are always coprime).

---

### Exploit Execution
The following Python script automates the process of connecting to the server, parsing the totient, and sending the calculated exponent:

```python
import socket, re, time

# Connection details
s = socket.socket()
s.connect(('enigma.aws.jerseyctf.com', 9001))
s.settimeout(10)

# Receive data until totient is provided
data = b''
while b'totient is ' not in data:
    data += s.recv(4096)

# Extract totient using regex
tot_n = int(re.search(r'totient is (\d+)', data.decode()).group(1))

# Apply Euler's Theorem: e = phi(n) + 1
e = tot_n + 1
s.send(str(e).encode() + b'\n')

# Retrieve the flag
time.sleep(2)
resp = b''
try:
    while True:
        chunk = s.recv(4096)
        if not chunk: break
        resp += chunk
except: pass

print(resp.decode())
s.close()
```

### Final Result
Upon sending the calculated exponent, the server confirms the match and provides the flag.

**Final Flag:**
`jctf5{ev1l_publ1c_k3ys_w1th_evil_public_d33ds}`
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
