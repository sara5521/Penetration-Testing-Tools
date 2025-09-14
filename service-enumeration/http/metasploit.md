# 🌐 Metasploit - Apache HTTP Enumeration

This document includes all Metasploit auxiliary modules used in the **Apache Enumeration** lab.

---

## 🎯 Purpose

This series of Metasploit modules allows you to enumerate different aspects of an Apache web server, including:

* HTTP version
* Server headers
* Directory and file discovery
* PUT and DELETE methods
* Login pages
* User directory enumeration

---

## 🧰 Tools & Modules Used

### 1. `http_version`

Detects the HTTP server version.

```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```
<img width="1508" height="520" alt="image" src="https://github.com/user-attachments/assets/f9bfa1ca-bbfb-42d2-b929-43d80233f204" />

### 2. `robots_txt`

Checks for `robots.txt` file.

```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```
<img width="1676" height="824" alt="image" src="https://github.com/user-attachments/assets/58ed561c-0537-43eb-ab9c-b3e0e1498c34" />

### 3. `http_header`

Grabs HTTP headers.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```
<img width="1676" height="666" alt="image" src="https://github.com/user-attachments/assets/d7125d08-92de-49a3-993a-ff19a3052fb5" />

### 4. `http_header` with URI

Grabs headers for a specific path.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```
<img width="1676" height="666" alt="image" src="https://github.com/user-attachments/assets/c817b731-83fc-4179-8e5a-931b86ec1b4f" />

### 5. `brute_dirs`

Brute-forces common directories.

```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```
<img width="1676" height="666" alt="image" src="https://github.com/user-attachments/assets/9e2c34c2-fe11-4af2-a0c6-070cdd44bae5" />

### 6. `dir_scanner`

Scans directories using a wordlist.

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```
<img width="2202" height="854" alt="image" src="https://github.com/user-attachments/assets/7c211137-dc4c-4cee-848a-b2af7caf20ea" />

### 7. `dir_listing`

Enumerates directory listing (like `/data`).

```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```
<img width="1654" height="570" alt="image" src="https://github.com/user-attachments/assets/f61eaed2-8633-4e4f-ba70-70e569680fd0" />

### 8. `files_dir`

Lists files inside a given directory.

```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```
<img width="1812" height="1248" alt="image" src="https://github.com/user-attachments/assets/de38b147-dbf5-4fac-9e14-020a36b85dd1" />
<img width="1812" height="1248" alt="image" src="https://github.com/user-attachments/assets/025de76a-5a53-44bb-b46e-8a55c489a341" />

### 9. `http_put`

Uploads a file using the PUT method.

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```
<img width="1812" height="654" alt="image" src="https://github.com/user-attachments/assets/8ae090af-b811-41ca-a52f-5ab29bd5ac74" />

### 10. `wget` to verify upload

```bash
wget http://victim-1:80/data/test.txt
cat test.txt
```
<img width="2656" height="724" alt="image" src="https://github.com/user-attachments/assets/312778f0-7238-4e86-866c-f55549509fa5" />

### 11. `http_put` to delete file

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run
```
<img width="1684" height="724" alt="image" src="https://github.com/user-attachments/assets/a5cce5e6-ec48-4170-b116-e95fe0ff4944" />

### 12. `wget` to confirm deletion

```bash
wget http://victim-1:80/data/test.txt
```
<img width="1684" height="524" alt="image" src="https://github.com/user-attachments/assets/5c3701f3-09a6-47da-bd62-4aa15bf56c0a" />

### 13. `http_login`

Attempts login brute-force.

```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```
<img width="1768" height="642" alt="image" src="https://github.com/user-attachments/assets/6a6f991b-6821-409b-9371-12f2221fd975" />

### 14. `apache_userdir_enum`

Enumerates user directories on Apache.

```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```
<img width="2628" height="1146" alt="image" src="https://github.com/user-attachments/assets/e9f14091-8369-4bd7-a7b3-ec1cb0cf3b0e" />

---

## 🧠 Pro Tips

* Combine `http_version`, `http_header`, and `robots_txt` to gather basic info quickly.
* Use `http_put` to test for file upload vulnerabilities.
* `apache_userdir_enum` helps you find user accounts served by Apache, useful for further enumeration or login attempts.

---

## 🔗 Related Modules

| Module Path                                  | Purpose                       |
| -------------------------------------------- | ----------------------------- |
| `auxiliary/scanner/http/http_version`        | Detect Apache HTTP version    |
| `auxiliary/scanner/http/robots_txt`          | Parse disallowed directories  |
| `auxiliary/scanner/http/http_header`         | Grab HTTP headers             |
| `auxiliary/scanner/http/http_login`          | Brute-force login credentials |
| `auxiliary/scanner/http/http_put`            | Upload or delete files        |
| `auxiliary/scanner/http/brute_dirs`          | Brute-force directories       |
| `auxiliary/scanner/http/files_dir`           | List files in a path          |
| `auxiliary/scanner/http/apache_userdir_enum` | Enumerate Apache userdirs     |

---

📚 [Metasploit Docs](https://docs.rapid7.com/metasploit/)
