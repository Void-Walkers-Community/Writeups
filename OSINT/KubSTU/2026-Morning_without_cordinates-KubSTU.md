# morningwithoutcoordinates

## Task
Identify the city and exact address from a single photo.

Flag format:
`KubSTU{CityName_StreetName_BuildingNumber}`

## Observations from the photo
- Large red rooftop sign reads **"АВТОВОКЗАЛ"** (bus station).
- A nearby sign reads **"АПТЕКА"**.
- Terrain is mountainous and coastal-looking, consistent with Black Sea resort cities.
- Story hint says they started near **Krasnoarmeyskaya Street**, which exists in many Russian cities, so this alone is not enough.

## Geolocation logic
1. The key anchor is the rooftop text: **АВТОВОКЗАЛ**.
2. Combined with coastal/mountain landscape, likely candidates are southern Russian Black Sea cities.
3. Cross-checking known bus stations in that region points strongly to **Gelendzhik**.
4. Public listings and references for **Автовокзал Геленджик** consistently give:
   - **г. Геленджик, ул. Объездная, 3**.
5. The building context (bus station + nearby pharmacy + terrain) matches this location.

## Final answer
City: **Gelendzhik**  
Address: **Obezdnaya 3**

## Flag
`KubSTU{Gelendzhik_Obezdnaya_3}`


---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


