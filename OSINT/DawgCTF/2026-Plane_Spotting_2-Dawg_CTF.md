# DawgCTF OSINT - Plane Spotting 2 Writeup

## Challenge
You are given an image of an approaching aircraft and asked:

> "What airport was it coming from?"

Flag format: `DawgCTF{IATA}`

---

## 1) Extract metadata from the image
Use EXIF to get the exact place/time of the photo:

```bash
exiftool /media/sf_kali/planespotting2.jpg
```

Key fields:
- `Date/Time Original`: `2026:04:05 15:22:55 -04:00`
- `GPS Latitude`: `39 deg 8' 24.52" N`
- `GPS Longitude`: `76 deg 38' 28.91" W`

This converts to approximately:
- `39.140144, -76.641364`

That location is in Glen Burnie, MD, directly under BWI arrival paths.

---

## 2) Identify the exact aircraft at that moment
Photo time in UTC:
- `2026-04-05 19:22:55 UTC`

Using ADS-B Exchange replay data for that half-hour chunk (`19:00-19:30 UTC`), parse aircraft near the EXIF coordinates at the nearest replay tick (`19:22:50 UTC`).

Closest aircraft to the camera location:
- Hex: `ABD1E3`
- Callsign metadata observed in replay: `SWA1868`
- Position at `19:22:50 UTC`: `39.142285, -76.64209`
- Altitude: ~`800 ft`

This matches the photographed final approach plane.

---

## 3) Resolve the origin airport for that flight
Query the callsign route:

```bash
curl -s 'https://api.adsbdb.com/v0/callsign/SWA1868'
```

Relevant result:
- Origin IATA: `NAS` (Lynden Pindling International Airport, Nassau)
- Destination IATA: `BWI`

So the plane in the image was coming from **NAS**.

---

## Final Flag

`DawgCTF{NAS}`

* [🔙 Back to OSINT Directory](../OSINT)
* [🔙 Back to OSINT Index Directory](../OSINT/INDEX.md)
* [🔙 Back to Main Directory](../README.md)
