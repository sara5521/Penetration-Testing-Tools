# Metasploit HTTP Enumeration Modules

This file contains documentation of the Metasploit auxiliary modules used in the **Apache Enumeration** lab. Each module helps enumerate HTTP services and content on a target machine running a web server.

---

## 1. `http_version`

**Module**: `auxiliary/scanner/http/http_version`

This module checks the HTTP service version running on the target.

```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```

✅ Output:

* Detected Apache 2.4.18 on Ubuntu.

---

## 2. `robots_txt`

**Module**: `auxiliary/scanner/http/robots_txt`

Checks for `robots.txt` file to discover disallowed or hidden paths.

```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```

---

## 3. `http_header`

**Module**: `auxiliary/scanner/http/http_header`

Used to view HTTP response headers from the target.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

---

## 4. `http_header` (custom path)

Scans a specific endpoint for HTTP headers.

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```

---

## 5. `brute_dirs`

**Module**: `auxiliary/scanner/http/brute_dirs`

Attempts to brute-force directory names on the target.

```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```

---

## 6. `dir_scanner`

**Module**: `auxiliary/scanner/http/dir_scanner`

Scans for common directories using a wordlist.

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

---

## 7. `dir_listing`

**Module**: `auxiliary/scanner/http/dir_listing`

Checks if directory listing is enabled on a given path.

```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```

---

## 8. `files_dir`

**Module**: `auxiliary/scanner/http/files_dir`

Lists files in directories where listing is enabled.

```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

---

## 9. `http_put` (upload file)

**Module**: `auxiliary/scanner/http/http_put`

Uploads a file to the web server using HTTP PUT.

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```

---

## 10. `wget` (download the uploaded file)

```bash
wget http://victim-1:80/data/test.txt
cat test.txt
```

---

## 11. `http_put` (delete file)

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run
```

---

## 12. `wget` (confirm deletion)

```bash
wget http://victim-1:80/data/test.txt
```

❌ Should return 404 Not Found.

---

## 13. `http_login`

**Module**: `auxiliary/scanner/http/http_login`

Attempts to brute-force login to a protected directory.

```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```

---

## 14. `apache_userdir_enum`

**Module**: `auxiliary/scanner/http/apache_userdir_enum`

Enumerates users based on Apache UserDir feature.

```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```
