Saber
_iamsaber
Creating global students unified movement, ISF

MrPS — Yesterday at 12:27 PM
There are unique flags for every team
And someone submitted the flag of other team
Saber — Yesterday at 12:34 PM
hey @MrPS @Q @7.thxxo who shared the flags !
Q [BURP],  — Yesterday at 12:34 PM
We are cooked
 [BURP], 
Saber — Yesterday at 12:34 PM
answer that !
Q [BURP],  — Yesterday at 12:35 PM
I not
MrPS — Yesterday at 12:35 PM
Don't know bro
Q is with me soo he didn't done that
@7.thxxo
Saber — Yesterday at 12:36 PM
deal sealed
no buddy talks about it now
MrPS — Yesterday at 12:36 PM
Tell something broo
Saber — Yesterday at 12:36 PM
we are clear now
Manager for a reason 😎
MrPS — Yesterday at 12:37 PM
Okay
Q [BURP],  — Yesterday at 12:37 PM
Can u explain
Why du mean
 [BURP], 
Saber — Yesterday at 12:38 PM
we are clear of flag sharing allegation
Q [BURP],  — Yesterday at 12:38 PM
Ohh
Saber — Yesterday at 12:39 PM
@Apex Protocol 
please share your writeups 
it must be human writeups !!!
i will check it once before you submit so please
Cyrus Isaac — Yesterday at 12:40 PM
the 2 forensic chall flag that i shared here are solved by Nova so he can give solution for them .....
Q [BURP],  — Yesterday at 12:42 PM
Incrementing Down, by doing initaial recon i found that it uses x last increment, header, so then , i got that it uses A simple bash loop that fires requests continuously and detects any page change:

PREV=""
while true; do
  CURR=$(curl -s http://311abb492a69fe7d.ctf.ac.upt.ro/)
  if [ "$CURR" != "$PREV" ] && [ -n "$PREV" ]; then
    echo "=== CHANGE DETECTED ==="
    echo "$CURR"
    PREV="$CURR"
  else
    PREV="$CURR"
  fi
done


This avoids processing every identical response and instantly surfaces the moment the counter rolls over and the flag appears.

When the counter hit 0, the <pre id="counter"> element was replaced with the flag in plaintext:

<pre id="counter">CTFAC{5548ed992a8e921067f9dcf836a0b82ac4939921f04a58f6111efe094fb4fce7}</pre>
 
 [BURP], 
Saber — Yesterday at 12:42 PM
challenge name
?
MrPS — Yesterday at 12:43 PM
Forwarded
babydroid
# babydroid

**Category:** Misc  
**Points:** 100  
**Author:** stancium  
**Solved on:** April 26, 2026  

message.txt
5 KB
Saber — Yesterday at 12:43 PM
human writeup
MrPS — Yesterday at 12:43 PM
Forwarded
cosmic # Cosmic

Target: http://c79ec57a8e72a87d.ctf.ac.upt.ro

Flag: CTFAC{21789fb2dab01d067f1df6174f184da6a66df2fd54f7c8ce7b98e8a4d0280f89}

Summary
The page is only a frontend. The real challenge is /captcha, which returns a large JSON object with 995 base64 JPEGs.

Each JPEG is a rendered token made from per-character glyph tiles with randomized noise/artifacts. A single capture is hard to read, but the underlying token stream is fixed across refreshes.

Solve path
Fetched /captcha several times and saved the responses.
Verified that token widths stay identical across refreshes, so the underlying token at each index is stable.
Averaged the same token position across 6 captures to suppress the visual noise.
Segmented each token into fixed-width character tiles.
Clustered the averaged character tiles to get a noisy substitution alphabet.
Ran a simple English n-gram hillclimber over the recovered token stream.
The decoded text contained a mid-log flag sequence written as words:

ctfac leftcurlybrace two one seven eight nine fb two dab hero one d hero sit seven f one df sit one seven four f one eight four da sit a sit sit df two fd five four f seven c eight ce seven b nine eight e eight a four d hero two eight hero f eight nine rightcurlybrace

Where:

hero -> 0
sit -> 6
leftcurlybrace -> {
rightcurlybrace -> }

That gives:

CTFAC{21789fb2dab01d067f1df6174f184da6a66df2fd54f7c8ce7b98e8a4d0280f89}

Validation
Submitting that exact string to /check returns {"result":"Correct"}.
Saber — Yesterday at 12:43 PM
human writeup
MrPS — Yesterday at 12:44 PM
Forwarded
# Pitlane Can Gateway

Author: thek0der

Points: 356

message.txt
6 KB
Saber — Yesterday at 12:44 PM
they want it 
not me 
i would have published them on github 
but they want it human writuep
MrPS — Yesterday at 12:45 PM
# 8 Bit Quest Writeup

## Challenge

- Name: `8 Bit Quest`
- Category: `Web`

message.txt
6 KB
MrPS — Yesterday at 12:46 PM
Give them too bro not have time too make it in humanoid form
btw they are good enough to be as human writeup
Saber — Yesterday at 12:46 PM
ohk
MrPS — Yesterday at 12:47 PM
# Another One Writeup

## Challenge

- Name: `Another One`
- Category: `Rev`

message.txt
7 KB
# Bank of AC Writeup

## Challenge

- Name: `Bank of AC`
- Category: `Misc`

message.txt
6 KB
# Chronomancer

Recovered key:

`CTFAC{13587d487718681377b6b43f539311e990edceaa796d23c8a4f12b8dbd72f105}`

message.txt
4 KB
# Crossing 33 Writeup

## Challenge

The attachment set contains:

message.txt
6 KB
# Domain Expansion

## Challenge

- Category: Web
- Points: 100

message.txt
6 KB
# Event Horizon Writeup

## Challenge

- Name: `Event Horizon`
- Category: `PWN`

message.txt
9 KB
# Firmware Whisper Writeup

## Challenge

- Name: `firmware whisper`
- Category: `REV`

message.txt
8 KB
# FrensFinance Writeup

## Challenge

- Name: `FrensFinance`
- Points: `100`

message.txt
7 KB
# NanoVault Ledger Writeup

## Challenge

- Name: `Nanovault Ledger`
- Points: `100`

message.txt
7 KB
# Phantomledger Writeup

## Challenge

- Name: `Phantomledger`
- Category: `Splunk / DFIR`

message.txt
8 KB
# Seven Minutes Writeup

## Challenge

- Name: `Seven Minutes`
- Category: `Crypto`

message.txt
6 KB
# Poly-Crypto Writeup

## Challenge

- Name: `Poly-Crypto`
- Category: `Crypto`

message.txt
5 KB
Saber — Yesterday at 3:27 PM
@Q last 2 writeups
Image
 [BURP], 
Saber — Yesterday at 3:27 PM
this is incrementing down
need writeup for its alive and imposter @Q
Q [BURP],  — Yesterday at 3:28 PM
solved by nova
 [BURP], 
Saber — Yesterday at 3:28 PM
both ?
Q [BURP],  — Yesterday at 3:28 PM
and @Cyrus Isaac
yes
he gave me
Saber — Yesterday at 3:28 PM
@Cyrus Isaac you have the writeup ?
@Nova0x ?
Cyrus Isaac — Yesterday at 3:29 PM
no bro forensic solved by nova
Saber — Yesterday at 3:29 PM
and @Q what about the osint ?
Q [BURP],  — Yesterday at 3:29 PM
idk who solved osint
 [BURP], 
Saber — Yesterday at 3:30 PM
you gave the flag 😂
Q [BURP],  — Yesterday at 3:30 PM
@Cyrus Isaac  gave me
read above chat
Cyrus Isaac — Yesterday at 3:31 PM
i told the solution for both
Saber — Yesterday at 3:31 PM
can you ping them once
i kind of lost em
Cyrus Isaac — Yesterday at 3:32 PM
ok
Saber — Yesterday at 3:33 PM
pitlane can gateway 
the lost ipod 
@MrPS these 2 ?
Image
Image
MrPS — Yesterday at 3:39 PM
The lost iPad is given by @Cyrus Isaac
Saber — Yesterday at 3:40 PM
and pitlane ?
Cyrus Isaac — Yesterday at 3:40 PM
its forensic one
Saber — Yesterday at 3:40 PM
yes
so nova right 😂
?
MrPS — Yesterday at 3:40 PM
Yah
😂
Wait I am giving u other one
Saber — Yesterday at 3:40 PM
nova --> Cyrus --> MRPS 
continue the chain
Saber — Yesterday at 3:40 PM
ohk
MrPS — Yesterday at 3:42 PM
# Pitlane Can Gateway

Author: thek0der

Points: 356

message.txt
6 KB
Cyrus Isaac — Yesterday at 7:03 PM
😭  i already told 2 forensic chall solve by himm
Saber — Yesterday at 7:06 PM
yeah yeah i have a goldfish memory so i forget everything 🙂
Nova0x — Yesterday at 7:57 PM
👀👀
Saber — Yesterday at 7:57 PM
writeups 🙂
Nova0x — Yesterday at 7:57 PM
Ok tonight ill send it 
Now im studying
Saber — Yesterday at 7:58 PM
ohk ohk
7.thxxo [@AC],  — Yesterday at 8:30 PM
I'm extremely sorry guys I had a urgent hackathon
won't let yall down next time 😔
Saber — 10:03 AM
@Nova0x writeups 🤡
Saber — 4:29 PM
REV (Reverse Engineering)
Another One
Chronomancer
Crossing 33
firmware whisper
babydroid

MISC
Bank of AC
cosmic
 
﻿
# Bank of AC Writeup

## Challenge

- Name: `Bank of AC`
- Category: `Misc`
- Points: `496`
- Author: `grdnero`

## Summary

This challenge presents a dead-looking GitHub Pages site:

- `https://bank-of-ac.github.io/`

The challenge text implies that the original branch vanished and that the public site no longer exposes the old vault content. The intended path is to treat the target as a recovery and web-archive problem rather than a conventional web exploitation task.

The key idea is that the live site is only a replacement layer. The original application still exists in the Internet Archive and contains the real flag directly in a client-side bundle.

## Step 1: Inspect the live site

The current site is still reachable, but it is clearly a themed decoy. It exposes several pages:

- `drawers/a.html`
- `drawers/b.html`
- `vaults/bronze.html`
- `vaults/silver.html`
- `vaults/gold.html`
- `ledger.html`

The visible clues include:

- `AC-VAULT-01`
- `Last balanced drawer: B`
- a notice about signing the ledger before requesting vault access

These clues suggest hidden content, but the live pages themselves do not reveal the flag.

## Step 2: Check for hidden paths

The next useful artifact is `robots.txt`:

```text
User-agent: *
Disallow: /records/
```

That strongly suggests the original site had additional material under `/records/`, even though the path now returns `404`.

At this point, the right pivot is historical recovery rather than more brute-force enumeration on the current deployment.

## Step 3: Query the Wayback Machine CDX index

Searching the Internet Archive for captures of `bank-of-ac.github.io` reveals an older snapshot:

```text
20260311172239 https://bank-of-ac.github.io/ 200 text/html
20260311172239 https://bank-of-ac.github.io/assets/index-Bk9wov-l.js 200 application/javascript
20260311172239 https://bank-of-ac.github.io/assets/index-jfNlZcxu.css 200 text/css
20260311172239 https://bank-of-ac.github.io/vite.svg 200 image/svg+xml
```

This is the critical breakthrough. The archived site is not the same as the current decoy pages. It is a Vite-built single-page application.

That immediately changes the approach:

- recover the archived `index.html`
- recover the archived JavaScript bundle
- inspect the client-side routes and embedded data

## Step 4: Load the archived application

The archived HTML is minimal:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>vault</title>
    <script type="module" crossorigin src="/assets/index-Bk9wov-l.js"></script>
    <link rel="stylesheet" crossorigin href="/assets/index-jfNlZcxu.css">
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

So the actual content is inside the archived JS bundle.

## Step 5: Inspect the archived JavaScript

After fetching and decompressing `index-Bk9wov-l.js`, the bundle can be searched for:

- routes
- hardcoded strings
- suspicious UI text
- flag-like values

The important route logic is straightforward:

- `/` renders a vault door UI
- clicking the button triggers navigation to `/secret`

The `"/secret"` route renders a shelf view inside the vault. In that component, the bundle contains a hardcoded code block:

```javascript
j.jsxs("code",{children:["CTF","{dc04c2197802d1375a3ddd78b66c0cf6b194ef645ecb657e505e555c9e121075}"]})
```

So the flag is not computed dynamically, decrypted in-browser, or fetched from an API. It is embedded directly in the archived client bundle.

## Step 6: Resolve the flag format discrepancy

The challenge statement says the flag format is `CTFAC{}`, but the recovered archived application clearly embeds:

```text
CTF{dc04c2197802d1375a3ddd78b66c0cf6b194ef645ecb657e505e555c9e121075}
```

That value was verified to be the accepted flag.

This means the format hint in the prompt is misleading or stale, while the archived application contains the actual intended answer.

## Flag

```text
CTF{dc04c2197802d1375a3ddd78b66c0cf6b194ef645ecb657e505e555c9e121075}
```

## Minimal Reproduction

The following commands are enough to reproduce the solve path:

```bash
curl -s https://bank-of-ac.github.io/robots.txt
curl -s 'https://web.archive.org/cdx/search/cdx?url=bank-of-ac.github.io/*&output=txt&fl=timestamp,original,statuscode,mimetype&filter=statuscode:200'
curl --compressed -sL 'https://web.archive.org/web/20260311172239id_/https://bank-of-ac.github.io/'
curl --compressed -sL 'https://web.archive.org/web/20260311172239id_/https://bank-of-ac.github.io/assets/index-Bk9wov-l.js' | rg 'CTF|secret|vault'
```

## Takeaway

This challenge is a clean example of archive-driven web recovery.

The live target was intentionally misleading, but it still exposed enough metadata to point toward older hidden content. Once the archived bundle was recovered, the rest of the solve reduced to ordinary static analysis of a client-side application.

When a CTF prompt emphasizes that a site used to exist and is now gone, historical infrastructure should be treated as part of the attack surface:

- `robots.txt`
- archived snapshots
- old static assets
- JavaScript bundles
- forgotten routes

In this case, the entire flag survived inside an archived SPA bundle long after the original branch had effectively disappeared.


---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
