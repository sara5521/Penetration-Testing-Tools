# Metasploit Modules for FTP Enumeration

This file contains useful Metasploit modules and commands for enumerating FTP services.

---

## 🔍 1. Check FTP Service Version

**Module:**
```
auxiliary/scanner/ftp/ftp_version
```

**Commands:**
```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target-ip>
run
```

**Purpose:**
- Identifies the FTP server version (e.g., ProFTPD, vsftpd, etc.)

---

## 🔐 2. FTP Brute Force Login

**Module:**
```
auxiliary/scanner/ftp/ftp_login
```

**Commands:**
```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target-ip>
set USER_FILE <path-to-userlist>
set PASS_FILE <path-to-passlist>
run
```

**Purpose:**
- Tries common username and password combinations to find valid FTP credentials.

📌 Default wordlists:
- `/usr/share/metasploit-framework/data/wordlists/common_users.txt`
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`

---

## 👤 3. Check for Anonymous FTP Login

**Module:**
```
auxiliary/scanner/ftp/anonymous
```

**Commands:**
```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target-ip>
run
```

**Purpose:**
- Checks if anonymous login is allowed (no username/password required)

---

## 💡 Notes

- You can run these modules directly from msfconsole.
- Always start with `ftp_version` to detect the service.
- Try anonymous login before brute force.
- Use `show options` to review module requirements.
