# Writeup

## Challenge Summary

We are given a VM image at `/media/sf_kali/ctf-vm.ova` and two hints:

1. Credentials are `root` with an empty password.
2. `history` does not print the real history file.

Flag format: `UMASS{}`

Final flag:

`UMASS{h3r35_7h3_c4rg0_vr00m}`

## Initial Observations

The second hint strongly suggests that interacting with the VM directly is misleading. Instead of trusting the shell, the safest approach is to inspect the disk image offline and read the actual files.

The OVA is just a tar archive:

```bash
file /media/sf_kali/ctf-vm.ova
```

This reveals:

- `ctf-vm.ovf`
- `ctf-vm.mf`
- `ctf-vm-file1.iso.gz`
- `ctf-vm-disk1.vmdk.gz`

The important artifact is the virtual disk: `ctf-vm-disk1.vmdk.gz`.

## Extracting the VM Disk

First extract the OVA and decompress the VMDK:

```bash
7z x /media/sf_kali/ctf-vm.ova -o/home/parth/HACK/ova_extract
gzip -dk /home/parth/HACK/ova_extract/ctf-vm-disk1.vmdk.gz
```

Then convert the VMDK to raw format for easier forensic inspection:

```bash
qemu-img convert -O raw \
  /home/parth/HACK/ova_extract/ctf-vm-disk1.vmdk \
  /home/parth/HACK/ova_extract/ctf-vm-disk1.raw
```

## Finding the Real Root Filesystem

Inspect the partition table:

```bash
mmls /home/parth/HACK/ova_extract/ctf-vm-disk1.raw
```

Relevant partitions:

- `2048` = `/boot`
- `3430400` = `/`

The root filesystem is the partition at offset `3430400`.

## Reading the Real Shell History

List root's home directory:

```bash
fls -o 3430400 /home/parth/HACK/ova_extract/ctf-vm-disk1.raw 131088
```

This reveals `.ash_history`, inode `131111`.

Read it directly:

```bash
icat -o 3430400 /home/parth/HACK/ova_extract/ctf-vm-disk1.raw 131111
```

Important commands from the real history:

```text
apk add rust
cargo install xor
...
git init .
...
echo "ajfesidpiunvzcoixuiuwjenfksdlzxjol" > ./*/red-herring
for f in $(find . -type d); do echo "kajdsfojczvioxjoij3" >> $f/red-herring; done
git commit -m "a bunch of nonsense"
git reflog
git log
...
ls 5457501C/
history
cat $HISFILE
```

This gives us the key idea:

- The user installed the Rust package `xor`.
- A git repo was created in `/home`.
- Filenames were transformed.
- There are planted `red-herring` files.

## Inspecting `/home`

Listing `/home` in the root filesystem shows many strange hex-like names:

```bash
fls -o 3430400 /home/parth/HACK/ova_extract/ctf-vm-disk1.raw 131081
```

One directory stands out:

`5457501C`

From the history we know the user ran:

```text
ls 5457501C/
```

That strongly suggests `5457501C` is meaningful.

## Recovering the XOR Key

The installed `xor` crate source was still present under `/root/.cargo`, so we can inspect how it works. Its recursive mode:

- XORs filenames with the key
- converts the result to hex

We also know many files are named `red-herring`, and their encrypted filename appears all over the tree:

`08555D451D131A075A5D0E`

Because XOR is reversible:

```text
key = encrypted_name XOR plaintext_name
```

Using:

- plaintext = `red-herring`
- ciphertext hex = `08555D451D131A075A5D0E`

we recover the first part of the key:

`z09huvhu33i`

Decrypting `5457501C` with that key gives:

`.git`

So the weird directory is actually the repository metadata directory.

## Using `.git` as a Crib

This is the turning point.

A freshly created `.git` directory contains very predictable filenames:

- `HEAD`
- `config`
- `description`
- `hooks/pre-commit.sample`
- `hooks/pre-push.sample`
- `info/exclude`
- etc.

By comparing the encrypted filenames in the VM against the known default names created by `git init`, we can recover much more of the key. Doing this yields a full repeating key with period 165:

```text
z09huvhu33i3bbuvuxzciohzcxviho3wryyudsfyuzcvxhyuhyuwrhyufdsuhhyuzvxcijlfkdasjknvoxzcihuwefijdsokncvlxznouhwe8dsoiljkcxnnnwue9fdsp8oicjxlvnbefhsoaduijkcvxnbywu9e8f0d9
```

## Decrypting File Contents

Filenames were XOR-encrypted and hex-encoded. File contents were XOR-encrypted directly.

For example, the encrypted git commit object did not decompress as zlib data until one XOR pass was applied with the recovered key.

After decrypting:

- `.git/refs/heads/master` contained the real commit hash
- `.git/logs/HEAD` showed the initial commit and stash activity
- `.git/refs/stash` pointed to the stash commit
- `.git/logs/refs/stash` contained the important message

## Where the Flag Was

Decrypting `.git/logs/refs/stash` produced:

```text
0000000000000000000000000000000000000000 55a10e0874b6d37a8b9c2d70468d91f5b8c78cf5 git stash <git@stash> 1774732415 +0000 On master: You found me! UMASS{h3r35_7h3_c4rg0_vr00m}
```

So the flag is:

`UMASS{h3r35_7h3_c4rg0_vr00m}`

## Why the Hints Matter

Hint 1:

- Root login is available, but using the live VM shell is misleading.

Hint 2:

- `history` is fake or manipulated.
- The real `.ash_history` on disk is trustworthy.

Without reading the disk directly, it is easy to get lost in the fake output and red herrings.

## Short Solve Path

1. Extract the OVA and decompress the VMDK.
2. Convert the VMDK to raw.
3. Use `mmls` to find the root partition offset `3430400`.
4. Read `/root/.ash_history` directly with `icat`.
5. Notice `cargo install xor`, `git init .`, and many `red-herring` writes.
6. Recover the XOR key from encrypted `red-herring` filenames.
7. Decrypt `5457501C` into `.git`.
8. Use default `git init` filenames to recover the full repeating key.
9. Decrypt git refs and logs.
10. Read `.git/logs/refs/stash` and extract the flag.

## Flag

`UMASS{h3r35_7h3_c4rg0_vr00m}`

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
