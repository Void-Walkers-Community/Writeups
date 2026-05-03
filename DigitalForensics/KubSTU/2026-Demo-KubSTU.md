# Demo Writeup

## Incident Summary
A web server compromise led to pivoting into the database server and theft of confidential data.

## Final Flag
`KubSTU{SQLi,shell.php,dbadmin,confidential_data.sql}`

## Step-by-Step Analysis

1. **Initial access vector: SQL Injection (SQLi)**
- File evidence: `service/var/www/html/index.php`
- Vulnerable code:
  - `$id = $_GET['id'];`
  - `$sql = "SELECT title, content FROM articles WHERE id = $id";`
- This is classic unsanitized SQL query construction.

2. **What was uploaded**
- Apache access log evidence (`service/var/log/apache2/access.log`):
  - `10:16:05` request shows SQLi with `INTO OUTFILE` writing:
    - `/var/www/html/uploads/shell.php`
  - Payload writes PHP webshell code using `system($_GET["cmd"])`.

3. **Post-exploitation activity on web server**
- Attacker executes:
  - `/uploads/shell.php?cmd=id`
  - `/uploads/shell.php?cmd=ls -la /home/www-data`
  - `/uploads/shell.php?cmd=cat /var/www/html/config.php`
- This indicates interactive command execution through uploaded webshell.

4. **Pivot to database server user**
- `config.php` contains DB host and SSH details, including:
  - `SSH_USER = dbadmin`
  - SSH key path `/home/www-data/.ssh_key_key`
- DB auth log (`DB/var/log/auth.log`) confirms:
  - `10:16:30` `Accepted publickey for dbadmin from 192.168.1.10`

5. **Under which user attacker operated**
- Confirmed user: `dbadmin`
- Further log evidence:
  - session opened for `dbadmin`
  - `sudo: dbadmin ... USER=root ; COMMAND=/bin/bash`

6. **What was copied**
- `DB/home/dbadmin/.bash_history` shows:
  - `cp /var/lib/mysql/confidential_data.sql /tmp/.backup_data`
- Therefore copied source file: `confidential_data.sql`

## Conclusion
- Vulnerability used: **SQL Injection**
- Uploaded file: **shell.php**
- User used after pivot: **dbadmin**
- Copied file: **confidential_data.sql**

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

