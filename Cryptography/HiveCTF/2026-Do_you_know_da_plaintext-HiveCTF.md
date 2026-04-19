**Writeup**

This challenge is a mix of Java reversing and a simple chained modular scheme.

The provided file was [`Main.class`](C:\Users\User\Downloads\Do_You_Know_Da_Plaintext\Main.class). Since it’s a compiled Java class, the first step is to inspect the embedded constants and reconstruct the logic.

From the bytecode, the important values are:

```text
p = 14480749232939546312395427075592596885726509716805118956959703103

ciphertexts = [
1125011765514190686712853473205855810354337155831400945424,
1476608676279858567166354524276373443128960,
1273936457427726191963474801400,
1948423663123674782171395845,
19873277368411206548353617,
56794441472517203955932840856421395948
]
```

The class also contains these methods:

- `encodeStr(String) -> BigInteger`
- `decodeBig(BigInteger) -> String`
- `decrypt_chunks(ArrayList<BigInteger>, BigInteger key, BigInteger p)`

### 1. Understanding the encoding

`encodeStr` converts a string into a `BigInteger` in little-endian base 256:

```java
res += char_value * 256^i
```

So a string like `"ABC"` becomes:

```text
65 + 66*256 + 67*256^2
```

`decodeBig` reverses this by repeatedly taking `% 256` and converting back to chars.

That means every decrypted chunk should turn into readable ASCII.

### 2. Understanding the crypto

The encryption logic is:

```java
c0 = m0 * key mod p
c1 = m1 * m0 mod p
c2 = m2 * m1 mod p
c3 = m3 * m2 mod p
...
```

And decryption is:

```java
m0 = c0 * key^-1 mod p
mi = ci * m(i-1)^-1 mod p
```

So the first chunk depends on the secret key, but every later chunk only depends on the previous plaintext chunk.

### 3. Recovering chunks

Because the flag format is known to be `HiveCTF{...}`, we can look for a chunk matching `HiveCTF{`.

Encoding `"HiveCTF{"` with the program’s base-256 format gives:

```text
8882879963476683080
```

Now test it against the ciphertext chain:

```text
c2 / encode("HiveCTF{") = 143414800455
```

That quotient decodes cleanly to:

```text
"G00d!"
```
So:

```text
m1 = "HiveCTF{"
m2 = "G00d!"
```

Then continue:

```text
c3 / encode("G00d!") = 13585931556171859 -> "S0Y0UD0"
c4 / encode("S0Y0UD0") = 1462783563 -> "KN0W"
c5 / encode("KN0W") = 38826278137852717966268835396 -> "D&P1A1NT3Xt}"
```

So the flag is:

```text
HiveCTF{G00d!S0Y0UD0KN0WD&P1A1NT3Xt}
```

### 4. Recovering the key

We can also recover the initial key-dependent chunk:

```text
m0 = "Password3#"
```

Using:

```text
c0 = m0 * key mod p
```

we get:

```text
key = c0 * m0^-1 mod p
    = 6767767676767676767767676767677677
```

### 5. Final answers

Flag:

```text
HiveCTF{G00d!S0Y0UD0KN0WD&P1A1NT3Xt}
```

Key:

```text
6767767676767676767767676767677677
```

Program output with the correct key:

```text
Password3#HiveCTF{G00d!S0Y0UD0KN0WD&P1A1NT3Xt}
```
---
* [🔙 Back to Cryptography Directory](../)
* [🔙 Back to Cryptography Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
