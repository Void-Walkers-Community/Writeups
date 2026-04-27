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
