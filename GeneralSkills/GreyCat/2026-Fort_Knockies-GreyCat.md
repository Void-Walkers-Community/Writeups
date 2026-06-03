Challenge name: Fort Knockies 

Step 1: Digging around the image
An OCI image is just a pile of gzipped tarballs plus some JSON manifests I listed the layers from manifest.json and used file on each blob to get a sense of sizes, most were tiny, a few were huge, the interesting ones were the small app-specific layers.
I extracted all the small layers and found:

ccde7c8 — /app/.env
5a09824 — /app/app.py, crypto.py, requirements.txt
d2c0088 — /app/templates/index.html
d8f983b — /var/lib/fortknockies/logs/fortknockies.log
3bf931e — /app/.git/ (a full git repo!)
67017f0 — /app/.wh..env (whiteout — the .env was deleted)
6d537ff — /app/.wh..git and /var/lib/fortknockies/.wh..staging (more whiteouts)

Whiteout files in OCI/Docker layers are how deletions work. A .wh.X file means "pretend X doesn't exist in the final image." But the previous layers still have the data. That's the whole point of this challenge.

Step 2: Reading the deleted files

The .env (from ccde7c8, before it was whiteout'd):
FLASK_ENV=production
UPLOAD_LIMIT=8388608
SEAL_FORMAT=FKENC1

pycache

app.py gave me the key line:
SEAL_PASSWORD = os.environ.get("FORTKNOCKIES_SEAL_KEY", "rookie-local-test-key")

So if FORTKNOCKIES_SEAL_KEY isn't set, it falls back to rookie-local-test-key.
crypto.py showed the current encryption format (FKENC1): AES-256-GCM with PBKDF2-HMAC-SHA256, 250k iterations.

The log:
2026-05-14T09:12:08Z INFO rookie upload sealer started
2026-05-14T09:12:19Z INFO sealed sample upload bytes=128
2026-05-14T09:13:02Z WARN cleanup pass removed local dev files

Step 3: The git repo
The .git directory in layer 3bf931e was a full repository I pointed GIT_DIR at it and ran git log:
cf02dfb remove legacy scratch files
446b2d6 add path mode test
e08083d keep legacy import notes
c5ee59e initial rookie test app

The commit e08083d added a file called scratch_crypto.py which was then deleted in cf02dfb, it contained the legacy decryption function, FKENC0:
def open_fkenc0(blob, password):
    # PBKDF2-HMAC-SHA1, 64000 iterations, AES-256-CBC
    
So there's an older encryption format.
Commit 446b2d6 added tests/test_parts.py with just:
part2 = "PATH"
And the final commit replaced it with:
# moved into local config during build testing

part2 = "PATH". So the password is assembled from parts.

Step 4: The hidden staging directory
The whiteout in 6d537ff deleted /var/lib/fortknockies/.staging, that means .staging existed somewhere before I found it in the same layer as the git repo (3bf931e):
/var/lib/fortknockies/.staging/README
I tried to cat it and got garbage, checked the magic bytes:
37 7a bc af ...
That's a 7z archive disguised as a README I extracted it with py7zr and got two files:

flag.enc
sample-upload.enc

flag.enc used FKENC0 the legacy format that was "deleted" from git. sample-upload.enc used FKENC1.

Step 5: Finding the password
Now I needed the password for FKENC0, the app falls back to rookie-local-test-key but that didn't work, the test file said part2 = "PATH" and was "moved into local config" meaning the key was split into parts and baked somewhere into the image config.
I went back to the .env that stray pycache line after the blank line what if that was part1 of the password? The test said part2 = "PATH", so the full password would be pycachePATH.
Tried it:
open_fkenc0(blob, "pycachePATH")
# -> b'grey{jz_some_rookie_mistakesi9v2k}\n'
Got it.

Flag: grey{jz_some_rookie_mistakesi9v2k}

Fort_Knockies_Writeup.txt
---
* [🔙 Back to General Skills Directory](../)
* [🔙 Back to General Skills Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
