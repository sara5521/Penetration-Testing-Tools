# Metasploit - HTTP Enumeration Modules (Detailed)

## Purpose
Detailed notes for Metasploit auxiliary modules used to enumerate HTTP/Apache services.  
Use this file for reconnaissance, finding hidden content, testing upload/delete functionality, and locating login forms.

## Key modules / usage
- `auxiliary/scanner/http/http_version` — detect web server and version.
- `auxiliary/scanner/http/robots_txt` — fetch robots.txt entries.
- `auxiliary/scanner/http/http_header` — collect HTTP headers.
- `auxiliary/scanner/http/brute_dirs` — quick directory brute force.
- `auxiliary/scanner/http/dir_scanner` — directory scanning with wordlist.
- `auxiliary/scanner/http/dir_listing` — check if directory listing is enabled.
- `auxiliary/scanner/http/files_dir` — search for common sensitive files.
- `auxiliary/scanner/http/http_put` — attempt HTTP PUT upload/delete.
- `auxiliary/scanner/http/http_login` — brute-force web login forms / basic auth.
- `auxiliary/scanner/http/apache_userdir_enum` — enumerate `~user` directories.

---

## Quick examples
Short commands you can run quickly in msfconsole.

```bash
# Start msfconsole
msfconsole -q

# Detect server/version
use auxiliary/scanner/http/http_version
set RHOSTS <target>
run

# Check robots.txt
use auxiliary/scanner/http/robots_txt
set RHOSTS <target>
run

# Scan headers for a custom path
use auxiliary/scanner/http/http_header
set RHOSTS <target>
set TARGETURI /secure
run

# Directory brute (quick)
use auxiliary/scanner/http/brute_dirs
set RHOSTS <target>
run

# Directory scan with wordlist
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target>
set DICTIONARY /path/to/directory_wordlist.txt
run

# Upload file using HTTP PUT
use auxiliary/scanner/http/http_put
set RHOSTS <target>
set PATH /uploads
set FILENAME test.txt
set FILEDATA "test"
run
```

---

## How to read common output
- `X: <value>` lines are headers. `Server:` header often shows server type and version.
- `200 OK` means request succeeded. `403` means forbidden. `404` not found.
- `Found /path/` or `exists (200 OK)` indicates a discovered resource.
- `Login Successful: user:pass` indicates valid credentials found.
- `Successfully uploaded file` / `Successfully deleted file` are clear success markers for PUT operations.

---

## Practical tips
- Use `http_version` first to profile the server. Map exact version to advisories.
- Run `robots_txt` and `http_header` before heavy directory brute forcing. They reduce blind guessing.
- Prefer targeted dictionaries for `dir_scanner` to reduce noise and runtime.
- Test `http_put` only in labs or with permission. Uploads can be destructive or trigger alerts.
- Keep timestamps and raw outputs. Save msfconsole logs: `spool` or redirect output.
- If you find credentials, test them on other services cautiously. Look for password reuse.

---

# Module details (expanded)

## http_version — Detect HTTP server and version

**Module**
```
auxiliary/scanner/http/http_version
```

**Purpose (short)**
Profile the web server type and version from banners and responses.

**Important options**
- `RHOSTS` — target host(s).
- `RPORT` — default 80.
- `SSL` — enable for HTTPS (or use port 443).
- `THREADS` — parallelism.

**What it does**
- Sends simple HTTP requests and inspects `Server` header and response body.
- Attempts to infer server product and version.

**Typical output**
```
[+] 192.0.2.3:80 Apache 2.4.18 ((Ubuntu))
```

**Interpretation**
- Use the version to search for known CVEs and vendor notes.
- Not definitive; banners can be modified or hidden.

**False positives / caveats**
- Server admin may mask or remove version headers.
- Reverse proxies can show proxy server version instead of backend.

**Follow-ups**
- Run `nmap` http scripts like `http-title`, `http-robots.txt`, and `http-enum`.
- Look up CVEs for the detected version.

**Evidence to collect**
- Raw HTTP response headers, msfconsole output, timestamp.

**Remediation**
- Hide or truncate version in headers.
- Patch the server and update to supported versions.

---

## robots_txt — Enumerate robots.txt entries

**Module**
```
auxiliary/scanner/http/robots_txt
```

**Purpose (short)**
Fetch `/robots.txt` to find disallowed paths which often reveal hidden or sensitive directories.

**Important options**
- `RHOSTS`, `RPORT`, `TARGETURI` (if server serves robots elsewhere).

**What it does**
- Retrieves `/robots.txt` and prints entries under `Disallow` and `Allow`.

**Typical output**
```
[*] /robots.txt found
[+] Contents of Robots.txt:
Disallow: /admin
Disallow: /backup
```

**Interpretation**
- `Disallow` entries often indicate admin or backup paths worth checking.
- Not proof of vulnerability. Check access permissions.

**Caveats**
- Not every disallowed path is valid or accessible.
- Some sites intentionally add noise to robots.txt to confuse scanners.

**Follow-ups**
- Attempt manual access to disallowed paths.
- Use `dir_scanner` or `brute_dirs` with focused wordlists.

**Evidence**
- Save robots.txt content and timestamp.

**Remediation**
- Do not expose sensitive paths in robots.txt.
- Use proper authentication and access controls.

---

## http_header — Grab HTTP headers

**Module**
```
auxiliary/scanner/http/http_header
```

**Purpose (short)**
Collect headers for fingerprinting and detecting protections or misconfigurations.

**Important options**
- `RHOSTS`, `RPORT`, `TARGETURI`.

**What it does**
- Requests the target URI and prints response headers like `Server`, `Set-Cookie`, `WWW-Authenticate`, `Content-Type`.

**Typical output**
```
:SERVER: Apache/2.4.18 (Ubuntu)
:CONTENT-TYPE: text/html
:WWW-AUTHENTICATE: Basic realm="Restricted Content"
```

**Interpretation**
- `WWW-AUTHENTICATE` indicates Basic auth.
- `Set-Cookie` may reveal session details or insecure flags.

**Caveats**
- Headers alone do not show full app behavior. Combine with body inspection.

**Follow-ups**
- If Basic auth detected, use `http_login` or targeted brute forcing.
- Check `Set-Cookie` flags (HttpOnly, Secure).

**Evidence**
- Save headers and full response.

**Remediation**
- Disable verbose server banners.
- Ensure cookies use Secure and HttpOnly flags. Use modern auth schemes.

---

## brute_dirs — Brute-force common directories

**Module**
```
auxiliary/scanner/http/brute_dirs
```

**Purpose (short)**
Quickly test common directory names with a small builtin list.

**Important options**
- `RHOSTS`, `RPORT`, `THREADS`.

**What it does**
- Tries a short list of common directories and reports found ones.

**Typical output**
```
[+] Found http://10.6.18.5:80/login/
```

**Interpretation**
- Found candidate directories. Investigate contents and access control.

**Caveats**
- Noisy and limited list. Use only for quick discovery.

**Follow-ups**
- Use `dir_scanner` with a larger custom list for deeper enumeration.

**Evidence**
- Save discovered URLs and response codes.

**Remediation**
- Remove or protect sensitive directories. Use auth and restrict indexing.

---

## dir_scanner — Directory scanner with wordlist

**Module**
```
auxiliary/scanner/http/dir_scanner
```

**Purpose (short)**
Scan directories using provided dictionary for deeper discovery.

**Important options**
- `DICTIONARY` — path to wordlist.
- `FOLLOW_REDIRECTS`, `THREADS`, `RHOSTS`.

**What it does**
- Iterates through wordlist paths and notes response codes and sizes.

**Typical output**
```
[+] 10.6.18.5:80/data/ exists (200 OK)
```

**Interpretation**
- `200 OK` indicates the path exists. `403` may mean restricted access.

**Caveats**
- Large lists are time-consuming. Respect lab rules and rate limits.

**Follow-ups**
- If path exists, check for listing, upload endpoints, or config files.

**Evidence**
- Wordlist used, timestamps, response codes.

**Remediation**
- Harden directories, disable directory listing, move backups outside web root.

---

## dir_listing — Directory listing check

**Module**
```
auxiliary/scanner/http/dir_listing
```

**Purpose (short)**
Check if directory index/listing is enabled and capture file list.

**Important options**
- `PATH`, `RHOSTS`.

**What it does**
- Requests the path and, if index not present, reports file entries.

**Typical output**
```
[+] Directory listing is enabled at /data/
 - file1
 - backup.zip
```

**Interpretation**
- Exposed files are a risk. Download and inspect safely.

**Caveats**
- Some servers show limited listings or generate dynamic listings.

**Follow-ups**
- Download interesting files with `wget` or `curl`.
- Search for credentials in config/backup files.

**Evidence**
- Directory listing output and file names.

**Remediation**
- Disable auto-indexing (`Options -Indexes` on Apache).
- Move backups out of webroot.

---

## files_dir — Search for sensitive files

**Module**
```
auxiliary/scanner/http/files_dir
```

**Purpose (short)**
Search for common sensitive filenames like backups and config files.

**Important options**
- `VERBOSE`, `DICTIONARY` (if applicable).

**What it does**
- Tests for known filenames and reports matches.

**Typical output**
```
[+] Found /config.php.bak
```

**Interpretation**
- Backup/config files may contain secrets. Prioritize inspection.

**Caveats**
- False positives possible if site has unrelated files with same name.

**Follow-ups**
- Download the file if allowed and inspect in a safe environment.

**Evidence**
- File path and content snapshot.

**Remediation**
- Remove backups from webroot, rotate credentials, and secure config files.

---

## http_put — Upload / Delete files via HTTP PUT

**Module**
```
auxiliary/scanner/http/http_put
```

**Purpose (short)**
Test if server accepts HTTP PUT for file upload or delete operations.

**Important options**
- `PATH`, `FILENAME`, `FILEDATA`, `ACTION` (UPLOAD/DELETE), `RHOSTS`.

**What it does**
- Performs PUT to create or DELETE to remove files at target path.

**Typical output**
```
[+] Successfully uploaded file: /data/test.txt
[+] Successfully deleted file: /data/test.txt
```

**Interpretation**
- Server supports write operations. Risk: remote file upload and code execution if webroot writable.

**Caveats**
- Some servers allow PUT but not execute. Others block certain extensions. Test carefully.

**Follow-ups**
- Check file access via `wget` or browser.
- If webroot writable, test uploading web shell only in lab and safe environment.

**Evidence**
- Uploaded file URL, file content, timestamps, and deletion logs.

**Remediation**
- Disable PUT where not needed. Restrict write directories and validate uploads.

---

## wget & cat — Confirm upload/download (CLI)

**Purpose**
Simple confirmation that uploaded files are reachable and to inspect contents.

**Commands**
```bash
wget http://<target>/data/test.txt
cat test.txt
```

**Typical output**
```
Welcome To AttackDefense
```

**Interpretation**
- File returned the expected content. Proof of write/read.

**Remediation**
- Monitor writeable directories and restrict access.

---

## http_login — Brute-force web login / Basic auth

**Module**
```
auxiliary/scanner/http/http_login
```

**Purpose (short)**
Attempt to authenticate against web forms or Basic auth using credential lists.

**Important options**
- `AUTH_URI` — login path for form or use `TARGETURI` for basic auth.
- `USER_FILE`, `PASS_FILE`, `USERPASS_FILE`.
- `METHOD` — form method if required (POST).
- `VERBOSE`, `STOP_ON_SUCCESS`.

**What it does**
- Submits credentials and looks for indicators of successful login (redirects, session cookies, specific body content).

**Typical output**
```
[+] 10.6.18.5:80 - Login Successful: admin:password123
```

**Interpretation**
- Found valid credentials. Use to access restricted functions.

**Caveats**
- False positives possible if page shows same response for failed login. Use multiple indicators.

**Follow-ups**
- After success, export session cookie and validate navigation to protected pages.

**Evidence**
- Request/response samples, cookies, successful lines.

**Remediation**
- Implement account lockouts, rate limiting, multi-factor auth.

---

## apache_userdir_enum — Enumerate Apache user directories

**Module**
```
auxiliary/scanner/http/apache_userdir_enum
```

**Purpose (short)**
Detect `~user` directories on Apache servers.

**Important options**
- `USER_FILE` — list of usernames to test.
- `RHOSTS`, `VERBOSE`.

**What it does**
- Tests common usernames and reports active `~user` directories.

**Typical output**
```
[+] Found userdir: http://10.6.18.5/~alice/
```

**Interpretation**
- Personal user directories may expose personal files or misconfigurations.

**Caveats**
- Many sites disable `userdir`. Results depend on server setup.

**Follow-ups**
- Inspect userdir content for credentials or sensitive files.

**Remediation**
- Disable `userdir` if unused. Apply proper permissions.

---

## INE lab example (complete scenario)

### Input
```bash
# Profile server
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run

# Get robots
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run

# Scan headers for /secure
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run

# Quick directory brute
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run

# Deeper directory scan
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/wordlists/directory.txt
run

# Check listing and files
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run

use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
run

# Try HTTP PUT upload
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run

# Confirm upload
!wget http://victim-1/data/test.txt
!cat test.txt

# Delete uploaded file
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run

# Brute-force login (if found)
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run

# Enumerate userdir
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

### Output (example aggregated)
```
[+] 192.74.12.3:80 Apache 2.4.18 ((Ubuntu))

[*] /robots.txt found
[+] Disallow: /data
[+] Disallow: /secure

[+] 192.74.12.3:80 :WWW-AUTHENTICATE: Basic realm="Restricted Content"

[+] Found http://10.6.18.5:80/login/
[+] 10.6.18.5:80/data/ exists (200 OK)
[+] Directory listing is enabled at /data/
    - test1.txt
    - backup.zip

[+] Found /config.php.bak
[+] Successfully uploaded file: /data/test.txt
Welcome To AttackDefense
[+] Successfully deleted file: /data/test.txt

[+] 10.6.18.5:80 - Login Successful: admin:password123
[+] Found userdir: http://10.6.18.5/~alice/
```

### Short analysis and next steps
- Server is Apache 2.4.18. Check vendor advisories.
- `/data` and `/secure` are interesting. `/data` has listing and backups; download and inspect `backup.zip` safely.
- `http_put` succeeded. If `/data` maps to webroot, test for web shell upload only in lab.
- Found `admin:password123`. Use session cookie to access `/secure/` and enumerate further.
- Userdirs may contain sensitive files. Document everything and include remediation notes.

---

## Final checklist (report)
- [ ] Save msfconsole raw output logs and timestamps.
- [ ] Save downloaded files and compute hashes.
- [ ] Note exact modules and options used.
- [ ] Provide remediation steps for each finding.
- [ ] Include safe reproduction steps for reviewers.

