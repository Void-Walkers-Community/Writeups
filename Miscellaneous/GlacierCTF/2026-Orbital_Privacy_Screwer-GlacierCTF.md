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

This was an important finding: the supplied source calculated progress using cumulative event durations, but the observed live service produced uniform percentage increments.

We abandoned percentage-based decoding and focused entirely on event timing.

## 6. Second Attempt — Timing-Based Decoding

We recorded the arrival time of each SSE event using Python's `time.perf_counter()`.

For consecutive events, we calculated:

`duration_ms = (next_timestamp - current_timestamp) * 1000`

We then classified the symbols using a threshold of 100 milliseconds:

- Duration below 100 ms → `.`
- Duration above 100 ms → `-`
- `carrier = false` → Character separator

This produced a nearly recognizable message beginning with `GCTF`, but several characters were incorrect.

The problem was network jitter. Internet latency caused some event durations to deviate from their expected 67 ms or 133 ms values.

A single measurement was not sufficiently reliable.

## 7. Final Solution — Multiple Captures and Median Timing

To eliminate timing noise, we developed `orbital_multi_solver.py`.

The solver performed the following operations:

1. Connected to the GlacierCTF launcher and started an instance.
2. Waited until `/api/health` returned `ONLINE`.
3. Opened `/api/acquire` and captured all 67 events.
4. Repeated the acquisition **nine times**, using separate request and session identifiers.
5. Collected the duration of each event across all nine runs.
6. Calculated the median duration for each event index.
7. Classified dots and dashes using the median timings.
8. Split characters at separator events and decoded the Morse sequence.

We used the median rather than the mean because it is less sensitive to occasional large network delays.

The important decoding logic was:

`median_duration[i] = median(duration_from_each_capture[i])`

`symbol = "-" if median_duration[i] >= 100 else "."`

We also implemented an independent verification method that classified the 22 longest active signals as dashes, matching the expected dash count in the source.

Both approaches recovered the same message.

## 8. Successful Results

After nine acquisitions, the solver reported:

| Measurement | Result |
|---|---:|
| Successful captures | 9 |
| Events per capture | 67 |
| Active carrier events | 52 |
| Character separators | 15 |
| Reconstructed dots | 30 |
| Reconstructed dashes | 22 |

The final Morse transmission was:

```text
--. / -.-. / - / ..-. / ----- / -... / .-.. / .. / ...- / - / -.. / .-. / ...-- / ... / --... / -
```

Decoding each token produced:

```text
G C T F 0 B L I V T D R 3 S 7 T
```

Both the 100 ms threshold method and the duration-ranking method agreed:

```text
GCTF0BLIVTDR3S7T
```

The result also matched the source's expected dot, dash, separator, and total-event counts.

After converting the decoded text to lowercase, we obtained the flag.

## 9. Flag

**`gctf0blivtdr3s7t`**

The value was recovered from the live server's timing data and independently verified using two decoding methods. The challenge platform's submission response was not captured in this writeup.

## 10. Conclusion

Orbital Privacy Screwer demonstrates how sensitive information can leak through timing even when the data itself is hidden from API responses.

The application encoded its flag using Morse code and transmitted the information through predictable delays between SSE events.

Although the response fields did not explicitly reveal the flag, the timing differences provided enough information to reconstruct the complete message.

Network jitter initially caused incorrect decoding, but repeating the transmission nine times and calculating median event durations produced a reliable result.

**Key takeaway:** Hiding information from response bodies does not prevent disclosure when secret-dependent behavior is observable through timing. This challenge was solved using side-channel analysis, repeated measurements, and Morse decoding.

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
