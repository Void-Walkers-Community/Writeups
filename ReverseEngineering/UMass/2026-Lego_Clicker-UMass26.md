# Lego Clicker Writeup

## Challenge

Hackers have taken over and corrupted your beloved Lego Clicker game, can you reclaim the top of the leaderboard?

Flag format: `UMASS{}`

## Flag

`UMASS{br1ck_by_br1ck_y0u_r3ach3d_th3_t0p}`

## Solve Summary

The zip contains a single Android APK:

- `LegoClicker_umass.apk`

After decompiling with `jadx` and `apktool`, the important logic shows up in the leaderboard activity and a native library:

- Java side: `com.example.LegoClicker.RA`
- Native side: `lib/*/liblegocore.so`

The app has obvious fake-flag bait in popup-style flows, but the real flag path is tied to the leaderboard and a native validator.

## Key Reversing Notes

### 1. The leaderboard is the real target

In `RA.java`, when the player is at the top of the leaderboard, the app calls:

- `SessionValidator.validateBrickToken(j, j)`
- `SessionValidator.a(j, j)`

If both succeed, the result is displayed as the reward text.

### 2. The real JNI methods are hidden

`SessionValidator` declares:

- `refreshTileMap(long, long)`
- `syncBrickCache(long, long)`
- `validateBrickToken(long, long)`

But these are not directly obvious from Java alone. In `JNI_OnLoad` inside `liblegocore.so`, the library dynamically registers the real native methods.

By decoding the JNI registration strings, we get:

- Class: `com/example/LegoClicker/SessionValidator`
- Methods:
  - `syncBrickCache`
  - `refreshTileMap`
  - `validateBrickToken`
- Signature: `(JJ)Ljava/lang/String;`

### 3. Anti-debug / anti-Frida checks exist

The native library also decodes and checks:

- `/proc/self/status` for `TracerPid:`
- `/proc/self/maps` for:
  - `frida`
  - `gadget`
  - `gum-js`
  - `linjector`

Those branches are there to feed decoys or alternate results. The clean-runtime branch is the one we care about.

### 4. The real flag comes from `syncBrickCache`

The native function used by `SessionValidator.a(...)` resolves to the hidden JNI handler for `syncBrickCache`.

Once the branch logic and byte transforms are unwound correctly, the decoded output is:

`UMASS{br1ck_by_br1ck_y0u_r3ach3d_th3_t0p}`

## Practical Workflow

1. Unzip the archive.
2. Decompile the APK with `jadx` and `apktool`.
3. Inspect `RA.java` and `SessionValidator.java`.
4. Follow `JNI_OnLoad` in `liblegocore.so`.
5. Decode the obfuscated JNI registration strings.
6. Ignore fake reward paths and analyze the clean branch of `syncBrickCache`.
7. Recover the decoded final string.

## Useful Commands

```bash
unzip -o /media/sf_kali/lego-clicker.zip -d /home/parth/HACK/lego-clicker
jadx -d /home/parth/HACK/lego-clicker/jadx /home/parth/HACK/lego-clicker/LegoClicker_umass.apk
apktool d -f /home/parth/HACK/lego-clicker/LegoClicker_umass.apk -o /home/parth/HACK/lego-clicker/apktool
rg -n "leaderboard|SessionValidator|UMASS|validateBrickToken|syncBrickCache" /home/parth/HACK/lego-clicker/jadx /home/parth/HACK/lego-clicker/apktool
```

## Final Answer

`UMASS{br1ck_by_br1ck_y0u_r3ach3d_th3_t0p}`



---
* [🔙 Back to Reverse Engineering Directory](../)
* [🔙 Back to Rev Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)



