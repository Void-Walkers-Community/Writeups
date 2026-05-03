Access chain reconstructs as:
COOPER R. (10.10.10.10) -> Switch0 Fa0/1 -> trunk on Fa0/24 -> Mastif_0828 router-on-a-stick with Gi0/0.10 = 10.10.10.1 and Gi0/0.20 = 10.20.20.1 -> Server#1 (10.20.20.100).

On the server, the useful path is HTTP:
index.html -> hint to copyrights.html in the legal directory.

The planted key string there is end_user_license_agreement, so the flag is:

KubSTU(end_user_license_agreement)

---
* [🔙 Back to Cloud Security Directory](../)
* [🔙 Back to Cloud Security Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
