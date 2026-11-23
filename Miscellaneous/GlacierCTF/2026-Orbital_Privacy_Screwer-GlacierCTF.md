# Orbital Privacy Screwer — GlacierCTF 2026 Writeup

**Challenge:** Orbital Privacy Screwer  
**Category:** Miscellaneous  
**Difficulty:** 50 points  
**Author:** Kobalsky  
**Connection:** `nc challs.glacierctf.com 13381`  
**Flag:** `gctf0blivtdr3s7t`

## 1. Challenge Description

> You better hide well, and you do it fast, because otherwise we will find you, and if we have to go to orbit for it, we will.

The challenge provides a surveillance-themed web application that simulates satellite target acquisition. Users can submit intelligence queries and watch a progress bar update during the acquisition process.

Our objective was to discover how the application leaked its hidden flag.

## 2. Initial Analysis

After extracting the provided `orbital-privacy-screwer.tar.gz` archive, we found the Python backend (`server.py`) and JavaScript frontend (`app.js`).

The backend implemented an API endpoint:

`GET /api/acquire`

This endpoint uses **Server-Sent Events (SSE)** to continuously stream progress updates to the browser.

Examining `server.py`, we discovered a Morse-code encoder that converts the flag into a sequence of dots, dashes, and character separators.

The source defined three durations:

| Symbol | Duration |
|---|---:|
| Dot (`.`) | 67 ms |
| Dash (`-`) | 133 ms |
| Character separator | 200 ms |

It also defined the expected transmission parameters:

- 30 dots
- 22 dashes
- 15 separators
- 67 total events
- 7,936 milliseconds total duration

This suggested that the flag was transmitted through the timing of the acquisition updates rather than through the application's visible diagnostic responses.

## 3. Understanding the Vulnerability

The backend constructed a Morse sequence from the flag and transmitted SSE messages containing fields such as:

- `type`: Event type, such as `step` or `complete`
- `seq`: Sequence number
- `pct`: Acquisition progress percentage
- `carrier`: Whether the carrier signal is active

The crucial code in `server.py` was the acquisition loop, which sent an event and then slept for its encoded duration:

`self.send_sse_message(payload)`

`precise_sleep(event_data["duration_ms"] / 1000.0)`

Although the SSE response did not directly reveal the Morse symbol, the elapsed time between consecutive events corresponded to the encoded symbol's duration.

This created a **timing side-channel vulnerability**.

An attacker could observe the event timing and reconstruct the hidden Morse message without accessing the flag file.

## 4. Launching the Challenge Instance

Connecting directly to the provided service displayed:

`Press [ENTER] to start a new instance`

After sending a newline, the launcher returned a temporary HTTPS address ending in:

`.orbital-privacy-screwer.web.glacierctf.com`

The launcher connection had to remain open while interacting with the web application. Closing the connection would stop the instance.

Once the temporary server started, its readiness was verified through:

`GET /api/health`

A successful response contained:

`{"status":"ONLINE"}`

We then connected to `/api/acquire`, supplying a query, request ID, and session ID.

## 5. First Attempt — Percentage-Based Decoding

Our initial approach used changes in the `pct` field to distinguish Morse symbols.

However, the live server returned the same percentage increase for every event:

`100 / 67 = 1.492537%`

Therefore, percentage differences contained no usable Morse information.

... (103 lines left) 

