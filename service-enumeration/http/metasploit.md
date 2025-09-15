# 📦 Metasploit - HTTP Enumeration Modules

This file explains useful **Metasploit auxiliary modules** for HTTP/Apache service enumeration.  

---

## 🎯 Purpose

These modules help you:
- Detect HTTP server version & headers  
- Enumerate `robots.txt` and hidden directories  
- Discover directory listings and sensitive files  
- Upload / delete files via HTTP PUT  
- Brute-force web login forms  
- Enumerate Apache user directories  

---

## 🗂️ HTTP Enumeration Modules Summary

| #  | Module                       | What it Does                           |
|----|-------------------------------|----------------------------------------|
| 1  | http_version                  | Detect web server version              |
| 2  | robots_txt                    | Enumerate disallowed entries in robots.txt |
| 3  | http_header                   | Grab HTTP headers                      |
| 4  | http_header (with TARGETURI)  | Grab HTTP headers from custom path     |
| 5  | brute_dirs                    | Brute-force common directories         |
| 6  | dir_scanner                   | Scan directories using wordlist        |
| 7  | dir_listing                   | Detect if directory listing is enabled |
| 8  | files_dir                     | Find sensitive files                   |
| 9  | http_put                      | Upload files to the server             |
| 10 | wget & cat                    | Confirm file upload/download           |
| 11 | http_put (delete)             | Delete uploaded file                   |
| 12 | wget (after delete)           | Confirm file deletion                  |
| 13 | http_login                    | Brute-force web login page             |
| 14 | apache_userdir_enum           | Enumerate Apache user directories      |

---

## 1️⃣ Detect HTTP Version

**Module**
```bash
auxiliary/scanner/http/http_version
```

**Command**
```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] 192.74.12.3:80 Apache 2.4.18 ((Ubuntu))
```

🔍 **Interpretation:**
- The web server is **Apache 2.4.18** running on Ubuntu.  
- Useful for checking known vulnerabilities against the exact version.  

---

## 2️⃣ Enumerate robots.txt

**Module**
```bash
auxiliary/scanner/http/robots_txt
```

**Command**
```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[*] [192.74.12.3] /robots.txt found
[+] Contents of Robots.txt:
# robots.txt for attackdefense
User-agen: test
# Directories
Allow: /webmail

User-agent: *
# Directories
Disallow: /data
Disallow: /secure
```

🔍 **Interpretation:**
- Found **hidden directories** `/data` and `/secure`.  
- These may contain sensitive files or restricted areas.  

---

## 3️⃣ Grab HTTP Headers

**Module**
```bash
auxiliary/scanner/http/http_header
```

**Command**
```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] 192.74.12.3:80        :CONTENT-TYPE: text/html
[+] 192.74.12.3:80        :LAST-MODIFIED: Wed, 27 Feb 2019 04:21:01 GMT
[+] 192.74.12.3:80        :SERVER: Apache/2.4.18 (Ubuntu)
[+] 192.74.12.3:80        :detected 3 headers
```

🔍 **Interpretation:**
- **Server:** Apache 2.4.29 (Ubuntu)  
- **Backend Language:** PHP 7.2.24  
- **Cookie:** PHPSESSID session cookie — might be useful for hijacking or testing authentication.  

---

## 4️⃣ Grab HTTP Headers (Custom Path)

**Command**
```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```

📸 **Sample Output:**
```
[+] 192.74.12.3:80        :CONTENT-TYPE: text/html; charse=iso-8859-1
[+] 192.74.12.3:80        :SERVER: Apache/2.4.18 (Ubuntu)
[+] 192.74.12.3:80        :WWW-AUTHENTICATE: Basic realm="Restricted Content"
[+] 192.74.12.3:80        :detected 3 headers
```

🔍 **Interpretation:**
- The `/secure` path requires **HTTP Basic Authentication**.  
- Useful for brute-forcing credentials with `http_login`.  

---

## 5️⃣ Brute-force Directories

**Command**
```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] Found http://10.6.18.5:80/images/
[+] Found http://10.6.18.5:80/login/
```

🔍 **Interpretation:**
- Discovered directories: `/images/` and `/login/`.  
- `/login/` is a good candidate for brute-force login attempts.  

---

## 6️⃣ Directory Scanner with Wordlist

**Command**
```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80/data/ exists (200 OK)
```

🔍 **Interpretation:**
- Found `/data/` directory.  
- May allow file upload/download (check with `http_put` or `dir_listing`).  

---

## 7️⃣ Directory Listing Check

**Command**
```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```

📸 **Sample Output:**
```
[+] Directory listing is enabled at /data/
    - test1.txt
    - backup.zip
```

🔍 **Interpretation:**
- Directory listing enabled → you can browse and download files directly.  
- Sensitive files (`backup.zip`) are exposed.  

---

## 8️⃣ Search for Sensitive Files

**Command**
```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

📸 **Sample Output:**
```
[+] Found /phpinfo.php
[+] Found /config.php.bak
```

🔍 **Interpretation:**
- `phpinfo.php` → may reveal server configuration.  
- `config.php.bak` → backup file, often contains database credentials.  

---

## 9️⃣ Upload File via HTTP PUT

**Command**
```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```

📸 **Sample Output:**
```
[+] Successfully uploaded file: /data/test.txt
```

🔍 **Interpretation:**
- Server allows **HTTP PUT** method.  
- File uploaded successfully, meaning write access is possible.  

---

## 1️⃣0️⃣ Confirm Upload with wget

**Command**
```bash
wget http://victim-1:80/data/test.txt
cat test.txt
```

📸 **Sample Output:**
```
Welcome To AttackDefense
```

🔍 **Interpretation:**
- Confirms file was uploaded and is accessible.  

---

## 1️⃣1️⃣ Delete File via HTTP PUT

**Command**
```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run
```

📸 **Sample Output:**
```
[+] Successfully deleted file: /data/test.txt
```

🔍 **Interpretation:**
- The server allows file deletion over HTTP PUT.  

---

## 1️⃣2️⃣ Confirm File Deletion

**Command**
```bash
wget http://victim-1:80/data/test.txt
```

📸 **Sample Output:**
```
404 Not Found
```

🔍 **Interpretation:**
- Confirms the uploaded file was deleted successfully.  

---

## 1️⃣3️⃣ HTTP Login Brute-force

**Command**
```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80 - Login Successful: admin:password123
```

🔍 **Interpretation:**
- Found valid credentials → `admin:password123`.  
- Can now access the restricted area `/secure/`.  

---

## 1️⃣4️⃣ Apache Userdir Enumeration

**Command**
```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

📸 **Sample Output:**
```
[+] Found userdir: http://10.6.18.5/~alice/
[+] Found userdir: http://10.6.18.5/~bob/
```

🔍 **Interpretation:**
- Apache UserDir module is enabled.  
- Each user has a personal directory (`/~alice/`, `/~bob/`).  
- These often contain personal files or misconfigured permissions.  
