# Writeup

## Challenge Summary

We are given a Java game archive:

- `/media/sf_kali/PacManForCTF.jar`

The game claims there is a high score of `6942069`, but normal gameplay makes that impossible. The goal is to recover the flag.

## Initial Recon

First, identify the file and inspect the archive contents:

```bash
file /media/sf_kali/PacManForCTF.jar
jar tf /media/sf_kali/PacManForCTF.jar
```

Relevant classes:

- `SimplePacMan.class`
- `JTextBasket.class`

The manifest shows the entry point is `SimplePacMan`:

```bash
unzip -p /media/sf_kali/PacManForCTF.jar META-INF/MANIFEST.MF
```

## Reverse Engineering

I used `javap` to inspect the bytecode:

```bash
mkdir -p /home/parth/HACK/hacman_jar
cd /home/parth/HACK/hacman_jar
jar xf /media/sf_kali/PacManForCTF.jar

javap -classpath . -c -p SimplePacMan > /home/parth/HACK/SimplePacMan.javap.txt
javap -classpath . -c -p JTextBasket > /home/parth/HACK/JTextBasket.javap.txt
```

## Why The High Score Is Impossible

Inside `SimplePacMan.actionPerformed`, the score logic is:

- every new pellet adds `10` points
- if score reaches `64000`, the game sets `loser = true`
- if score is ever `>= 6942069`, the game sets `winner = true`

So the game is intentionally unwinnable through normal play. You lose long before the displayed high score.

In pseudocode, the important part looks like this:

```java
if (score >= 6942069) {
    winner = true;
    score = 6942069;
}

// normal movement and pellet collection...

score += 10;
if (score == 64000) {
    loser = true;
}
```

The in-game message even hints at this:

`In order to win, you need to cheat!`

## Where The Flag Comes From

When the win screen is drawn, the program does something unusual:

1. It sets the panel name to the score.
2. It calls `revalidate()` on a hidden component.
3. That hidden component (`JTextBasket`) uses the panel name as input.
4. It derives an AES key and IV from the score.
5. It decrypts a Base64 ciphertext.
6. The decrypted plaintext is used as the component name and displayed on screen.

The ciphertext hardcoded in `JTextBasket.revalidate()` is:

```text
6Ach6HiD0JmCc1L+RwxDRzhW3sC1kS6XydgSuWVFpxVXRU8EjfuMxIMoIzMwK/ii
```

## Decryption Logic

The code computes:

```text
n = (score * 10 + 1)^4
```

Then:

- AES key = bytes from `hex(n_as_decimal_string)`
- IV = bytes from `hex(reverse(decimal_string_of_n))`

For the forced winning score:

```text
score = 6942069
```

So:

```text
n = (6942069 * 10 + 1)^4
  = 23225000336468054454242927385361
```

That decimal string is treated as hex input by the Java code, and its reverse is used for the IV.

## Solve Script

This Node.js one-liner reproduces the game’s decryption exactly:

```bash
node -e "const crypto=require('crypto'); const score='6942069'; const n=(BigInt(score)*10n+1n)**4n; const keyHex=n.toString(); const ivHex=keyHex.split('').reverse().join(''); const key=Buffer.from(keyHex,'hex'); const iv=Buffer.from(ivHex,'hex'); const ct=Buffer.from('6Ach6HiD0JmCc1L+RwxDRzhW3sC1kS6XydgSuWVFpxVXRU8EjfuMxIMoIzMwK/ii','base64'); const decipher=crypto.createDecipheriv('aes-128-cbc',key,iv); let pt=decipher.update(ct); pt=Buffer.concat([pt,decipher.final()]); console.log(pt.toString('utf8'));"
```

You can also do it in Python:

```python
from base64 import b64decode
from Crypto.Cipher import AES

score = "6942069"
n = (int(score) * 10 + 1) ** 4

key_hex = str(n)
iv_hex = key_hex[::-1]

key = bytes.fromhex(key_hex)
iv = bytes.fromhex(iv_hex)
ct = b64decode("6Ach6HiD0JmCc1L+RwxDRzhW3sC1kS6XydgSuWVFpxVXRU8EjfuMxIMoIzMwK/ii")

pt = AES.new(key, AES.MODE_CBC, iv).decrypt(ct)
print(pt.decode().strip())
```

## Flag

```text
DawgCTF{ch3at3R_ch34t3r_pumk1n_34t3r!}
```

## Takeaway

This challenge is a nice example of:

- a deliberately impossible in-game objective
- hidden logic triggered only in the “win” state
- recovering the flag by reversing the client instead of playing the game

The key hint was that the game itself says cheating is required.

* [🔙 Back to Reverse Engineering Directory](../ReverseEngineering)
* [🔙 Back to Rev Index Directory](../ReverseEngineering/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
