1. Static S3 site leaked Cognito config in index.html
2. Cognito User Pool allowed public sign-up
3. New user could authenticate and receive an IdToken
4. IdToken was exchanged through Cognito Identity Pool for temporary AWS creds
5. Authenticated role could list/scan DynamoDB tables
6. admin-notes leaked vault_key = QUEEN-BEE-ALPHA
7. vault table blocked Scan but allowed GetItem with the partition key
8. GetItem returned the flag
Flag: HiveCTF{c0gn1t0_cr3d_v3nd1ng_m4ch1n3}
---
* [🔙 Back to Cloud Security Directory](../)
* [🔙 Back to Cloud Security Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
