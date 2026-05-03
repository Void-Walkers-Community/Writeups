# where is my faculty.md

## Challenge
The prompt says a faculty was previously called **IKSiIB**, later renamed, and asks for:
- current abbreviation
- the faculty “birthday”

Flag format:
`KUBSTU{Abbreviation_DD.MM.YYYY}`

## OSINT process
1. Searched for `IKSiIB` and `KUBSTU` references.
2. Found a relevant Telegram channel page:
   - **"Факультет информационных технологий и кибербезопасности (ФИТК) | КубГТУ"**
3. Extracted key lines from the page content:
   - It states that on **1 February 2014** the institute IKSiIB was created.
   - The current faculty branding is **ФИТК** (FITK).

## Reconstruction
- Current name/abbreviation: **FITK**
- Birthday date: **01.02.2014**

## Final flag
`KUBSTU{FITK_01.02.2014}`

---
* [🔙 Back to OSINT Directory](../)
* [🔙 Back to OSINT Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)


