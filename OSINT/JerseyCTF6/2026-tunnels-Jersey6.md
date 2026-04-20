## Tunnels

You're walking in the woods.

There's no one around and your phone is dead.

Out of the corner of your eye you spot it...

This awesome New Jersey tunnel!

The tunnel is named **x**. It is currently not in use. Very soon however, it will be back in use again to bring people west towards **y**.

After that, there are plans to potentially use this tunnel to bring people into Pennsylvania. An analysis of this plan was published in 2023. If following the schedule recommended in this analysis, it is estimated that **z** people will use this new service the fiscal year it opens and **w** people two fiscal years after.

**Flag Format:** `jctf{x,y,z,w}` 
**Example:** `jctf{Name_of_Tunnel, Place_Name, 123000,456000}`

---

2 images were given upon rev searching found out that **Roseville Tunnel** matches perfectly — a New Jersey tunnel currently not in use, being restored for NJ Transit service westward.

The prompt's key clues are: NJ tunnel, currently not in use, and will soon bring people west.

**$x=$ [Roseville Tunnel](https://en.wikipedia.org/wiki/Roseville_Tunnel)**

### Rehabilitation for Andover service
In 2011, New Jersey Transit received approval to re-lay 7.3 miles (11.8 km) of track from Port Morris Junction through the tunnel in order to open commuter rail service westward to **Andover, New Jersey**.

Eleven years passed before the next tangible step toward the work on Roseville Tunnel. On April 13, 2022, the NJ Transit Board of Directors approved a $32.5 million contract to Schiavone Construction Company of Secaucus, New Jersey. Under the contract, Schiavone will build 8,000 feet of track bed, strengthen and stabilize the tunnel interior, remove at least 15 to 20 feet of the tunnel (bringing the tunnel to just under 1,000 feet (300 m) in length), improve drainage, create an interior pedestrian path, and install a radio system and security cameras. It will also replace two culverts: the one at Hudson Farm, about 500 feet upstream from the Andover station site; and the Andover Junction Brook culvert at the future station. The work is to be completed within early 2025.

Restoration of the whole line to **Andover** is slated for completion in late 2026.

**$y=$ Andover**

---

**[Amtrak Analysis of Options - Scranton to New York](https://www.amtrak.com/content/dam/projects/dotcom/english/public/documents/corporate/reports/Analysis-of-Options-Scranton-New-York-Amtrak-Passenger-Rail-Service.pdf)**

**$z=302100$ & $w=473500$**

From Amtrak's March 2023 study on the proposed NYC-Scranton corridor (which would use this tunnel for the Pennsylvania extension), the recommended Option D schedule projects:

* **FY2028 (opening fiscal year):** 302,100 riders
* **FY2030 (two fiscal years after):** 473,500 riders

**Final Flag:** `jctf{Roseville_Tunnel, Andover, 302100,473500}`


---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
