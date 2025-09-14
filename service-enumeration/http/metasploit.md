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

| #  | Module                          | What it Does                           |
|----|---------------------------------|----------------------------------------|
| 1  | http_version                    | Detect web server version              |
| 2  | robots_txt                      | Enumerate disallowed entries in robots.txt |
| 3  | http_header                     | Grab HTTP headers                      |
| 4  | http_header (with TARGETURI)    | Grab HTTP headers from custom path     |
| 5  | brute_dirs                      | Brute-force common directories         |
| 6  | dir_scanner                     | Scan directories using wordlist        |
| 7  | dir_listing                     | Detect if directory listing is enabled |
| 8  | files_dir                       | Find sensitive files                   |
| 9  | http_put                        | Upload files to the server             |
| 10 | wget & cat                      | Confirm file upload/download           |
| 11 | http_put (delete)               | Delete uploaded file                   |
| 12 | wget (after delete)             | Confirm file deletion                  |
| 13 | http_login                      | Brute-force web login page             |
| 14 | apache_userdir_enum             | Enumerate Apache user directories      |

---

## 🌐 1. Detect HTTP Version
```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80 Apache httpd 2.4.29 ((Ubuntu))
```

🔍 **Interpretation:**
- The web server is **Apache 2.4.29** running on Ubuntu.  
- Useful for checking known vulnerabilities against the exact version.  

---

## 🤖 2. Enumerate robots.txt
```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80 Robots.txt found:
    Disallow: /admin/
    Disallow: /backup/
```

🔍 **Interpretation:**
- Found **hidden directories** `/admin/` and `/backup/`.  
- These may contain sensitive files or restricted areas.  

---

## 📰 3. Grab HTTP Headers
```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80 HTTP Headers:
    Server: Apache/2.4.29 (Ubuntu)
    X-Powered-By: PHP/7.2.24
    Set-Cookie: PHPSESSID=abcd1234; path=/
```

🔍 **Interpretation:**
- **Server:** Apache 2.4.29 (Ubuntu)  
- **Backend Language:** PHP 7.2.24  
- **Cookie:** PHPSESSID session cookie — might be useful for hijacking or testing authentication.  

---

## 📰 4. Grab HTTP Headers (Custom Path)
```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```

📸 **Sample Output:**
```
[+] 10.6.18.5:80/secure HTTP Headers:
    HTTP/1.1 401 Unauthorized
    Server: Apache/2.4.29 (Ubuntu)
    WWW-Authenticate: Basic realm="Restricted Area"
```

🔍 **Interpretation:**
- The `/secure` path requires **HTTP Basic Authentication**.  
- Useful for brute-forcing credentials with `http_login`.  

---

## 📂 5. Brute-force Directories
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

## 📑 6. Directory Scanner with Wordlist
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

## 📋 7. Directory Listing Check
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

## 📁 8. Search for Sensitive Files
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

## 📤 9. Upload File via HTTP PUT
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

## 📥 10. Confirm Upload with wget
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

## 🗑️ 11. Delete File via HTTP PUT
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

## 🔍 12. Confirm File Deletion
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

## 🔐 13. HTTP Login Brute-force
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

## 👤 14. Apache Userdir Enumeration
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
