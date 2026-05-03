# The City That Doesn't Exit

## Challenge
In one of the largest shopping centers in southern Russia, a children's themed park (licensed clone of a foreign concept) suddenly changed its name in 2019 without official statements.

Goal:
- Identify the managing legal entity
- Find the trademark application date in Rospatent
- Build flag as: `KubSTU{INN_date}`

## OSINT Path
1. The clue about a large southern-Russia mall + licensed foreign concept points to the children's city-of-professions format in Krasnodar.
2. In OZ Mall (one of the largest malls in southern Russia), media references connect the old branding `Minopolis` with later branding `ZkidZ city` around 2019.
3. Company registry data ties the park operation to:
   - `ООО "ТПКО МИНОПОЛИС КРАСНОДАР"`
   - INN: `2312196680`
4. In the official Rospatent/FIPS register entry for trademark `MINOPOLIS` (certificate №723853), the application filing date `(220)` is:
   - `20.12.2018`

## Final Flag
`KubSTU{2312196680_20.12.2018}`

## Sources
- https://www.yuga.ru/news/311645/
- https://www.yuga.ru/articles/society/9327.html
- https://checko.ru/company/tpko-minopolis-krasnodar-1122312010655
- https://www1.fips.ru/registers-doc-view/fips_servlet?DB=RUTM&DocNumber=723853&TypeFile=html


---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


