To keep the **Void-Walkers** archive consistent and professional, here are the two standard formats for your team members to follow.

---

## Option 1: External Writeup (Medium/Blog Link)
Use this if the writeup is long-form, contains many high-res images, or is intended for public branding on Medium.

**File Name:** `2026-Challenge_Name-Event_Name.md`

# [Category] | Challenge Name

> **Event:** Event Name 2026
> **Points:** XXX
> **Author:** @username

## 🔗 Official Writeup Link
You can find the detailed technical walkthrough for this challenge on our Medium publication:

### [Read the Full Writeup Here](https://medium.com/void-walkers/your-post-link)

---

## 🚩 Flag
`void{fl4g_h3r3}`

---

## 🗂️ Navigation
* [🔙 Back to Category Directory](../)
* [🔙 Back to Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

---

## Option 2: Internal GitHub Writeup (Full MD)
Use this for native repository documentation. This is the preferred method for internal technical knowledge sharing.
Start with path creation. Create a .md within the relevant sub-category (Format: YYYY-Challengename(Seperate using underscore)-EventName(Seperate using underscore)) (Eg. 2026-Lost_Biker-EHAX)
**File Path:** `[Category]/2026-Challenge_Name-Event_Name.md`

# 🛠️ [Category] | Challenge Name

## 📊 Challenge Overview
* **Event:** Event Name 2026
* **Difficulty:** [Easy / Medium / Hard]
* **Category:** [e.g., Web / Pwn / OSINT]
* **Points:** XXX
* **Solved By:** @username

### 📝 Description
> Paste the original challenge description here.

---

## 🕵️ Phase 1: Reconnaissance & Analysis
* **Initial Findings:** (e.g., "Found a hidden `/admin` endpoint" or "Detected a buffer overflow in the `input()` function").
* **Tools Used:** (e.g., Nmap, Burp Suite, Ghidra).
* **The Vulnerability:** Explain the "why" behind the bug.

---

## 🚀 Phase 2: Exploitation / Solution
1.  **Step One:** Describe the first technical action.
2.  **Step Two:** Describe the pivot or the bypass.
3.  **Step Three:** The final execution.

### 💻 Exploit Script / Commands
# Insert your solve.py or terminal commands here
import pwn

io = pwn.remote('addr', 1337)
# ... exploitation logic ...
io.interactive()

---

## 🏁 Conclusion
* **The Flag:** `void{fl4g_h3r3}`
* **Key Learning:** What did this challenge teach the team?

---

## 🗂️ Navigation
* [🔙 Back to Category Directory](../)
* [🔙 Back to Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

---

### 🛡️ Void-Walker Naming Convention Reminder
To keep the repo searchable, ensure every file follows the strict naming logic:
* **Format:** `YYYY-Challenge_Name-Event_Name.md`
* **Example:** `2026-Lost_Biker-EHAX.md`
* **Consistency:** Use underscores `_` for spaces and hyphens `-` to separate the Year, Name, and Event.
