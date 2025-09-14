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

### 3. `http_header`

Grabs HTTP headers.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

### 4. `http_header` with URI

Grabs headers for a specific path.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```

### 5. `brute_dirs`

Brute-forces common directories.

```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```

### 6. `dir_scanner`

Scans directories using a wordlist.

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

### 7. `dir_listing`

Enumerates directory listing (like `/data`).

```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```

### 8. `files_dir`

Lists files inside a given directory.

```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

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

### 10. `wget` to verify upload

```bash
wget http://victim-1:80/data/test.txt
cat test.txt
```

### 11. `http_put` to delete file

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run
```

### 12. `wget` to confirm deletion

```bash
wget http://victim-1:80/data/test.txt
```

### 13. `http_login`

Attempts login brute-force.

```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```

### 14. `apache_userdir_enum`

Enumerates user directories on Apache.

```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

---

## 📸 Sample Output

Sample outputs include:

* Apache/2.4.18 (Ubuntu) version detection
* Found directories or files
* Success/failure of file PUT/DELETE
* List of user directories (like `/~root`, `/~admin`, etc.)

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
