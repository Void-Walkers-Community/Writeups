## Writeup: Follow the Flock

The challenge **"Follow the Flock"** is an OSINT (Open Source Intelligence) task that requires identifying a specific location in Connecticut based on a set of environmental and sensory descriptions.

---

### **1. Analyzing the Clues**
The challenge provided the following text:
> *"Man these damn seagulls keep following me around… always in a flock, obnoxious sirens blaring. And they smell like pizza. Feels like I’m being watched everywhere I go…"*

* **The Pizza Connection:** The hint explicitly states the city is in Connecticut and is famous for its style of pizza. This immediately identifies **New Haven**, world-renowned for "New Haven-style apizza."
* **The "Flock" and Seagulls:** This suggests proximity to a large body of water or a specific landmark where seagulls congregate. In New Haven, this typically points toward the harbor or areas near the Quinnipiac and Mill Rivers.
* **Obnoxious Sirens:** This indicates a location near a major emergency route, hospital, or fire/police headquarters.
* **"Watched everywhere I go":** This is a clever reference to **State Street**. Not only is it a major thoroughfare, but it is also the location of **Modern Apizza**. The "watched" clue likely refers to the high density of traffic cameras and the urban, high-surveillance nature of this specific stretch compared to the more residential Wooster Street.

---

### **2. Narrowing the Location**
While **Wooster Street** is the most famous "pizza street" (home to Pepe's and Sally's), it is a relatively quiet, historic side street. It does not fit the "obnoxious sirens" or "watched everywhere" descriptions as well as **State Street**.

**State Street** is:
1.  Home to **Modern Apizza** (one of the "Big Three" pizzerias).
2.  A major transit artery where emergency vehicles (sirens) are constant.
3.  Located near the rail lines and parts of the city where seagulls are a permanent fixture.
---

### **3. Flag Construction**
* **Street Name:** State Street
* **City Name:** New Haven
* **State:** CT

**Flag:** `CIT{State_Street_New_Haven_CT}`

---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
