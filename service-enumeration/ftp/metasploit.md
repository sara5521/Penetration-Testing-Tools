---
tool: "ftp (metasploit)"
protocol: "ftp"
tags: ["ftp","metasploit","enumeration"]
level: "beginner"
date: "2025-09-16"
path: "service-enumeration/ftp/metasploit.md"
---

# 📦 Metasploit - FTP Enumeration Modules (Detailed)

## 📂 Path in GitHub Project
```
service-enumeration/ftp/metasploit.md
```

> ⚖️ **Ethics / Scope:** Run these checks only on systems you own or have explicit permission to test.

---

## 🎯 Purpose
Metasploit auxiliary modules to enumerate FTP services, test anonymous login, and perform credential checks.  
Use for reconnaissance, credential testing, and manual verification (download/upload) in labs.

---

## 🔗 Quick files (where outputs & screenshots go)
- Evidence (raw outputs): `service-enumeration/ftp/evidence/`  
- Screenshots: `service-enumeration/ftp/screenshots/`  
- Cheat-sheet (1-page): `service-enumeration/ftp/ftp-cheatsheet.md`  
- Flashcards CSV: `service-enumeration/ftp/ftp-flashcards.csv`

---

## 🗂️ Table of contents
- `ftp_version` — detect server and version  
- `ftp_login` — credential brute-force / credential testing  
- `anonymous` — anonymous access checks  
- `ftp (CLI)` — manual client usage and post-login actions

---

## 💡 Quick examples
```bash
# msfconsole examples
msfconsole -q

# Banner/version
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 10.10.10.5
run

# Brute-force credentials (careful)
use auxiliary/scanner/ftp/ftp_login
set RHOSTS 10.10.10.5
set USER_FILE /usr/share/wordlists/users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
run

# Anonymous check
use auxiliary/scanner/ftp/anonymous
set RHOSTS 10.10.10.5
run
```

---

## 🔎 How to read common output
- `FTP Server: ProFTPD 1.3.5a` → banner + version (map to CVEs)  
- `Login Successful: user:pass` → valid credentials (save in evidence only, DO NOT commit)  
- Directory listings → if uploads allowed, note path and test carefully in lab

---

## 🛠️ Practical tips
- Save raw output into `service-enumeration/ftp/evidence/<tool>_s<step>_output.txt`.  
- For brute-force use slow threads and `STOP_ON_SUCCESS` to reduce noise.  
- If anonymous allowed: manually `ftp` to confirm `ls`, `get`, `put` (only in lab).  
- Add nmap `-oA` xml to evidence for reproducibility.

---
# 🔍 Module details (expanded)

### ftp_version — Detect FTP server and version
**Module**
```
auxiliary/scanner/ftp/ftp_version
```

**Important options**
- `RHOSTS`, `RPORT` (21), `THREADS`  
**Typical output (snippet)**  
```
[+] 192.0.2.3:21 - FTP Server: ProFTPD 1.3.5a
```  
**Evidence (save full output):**  
`service-enumeration/ftp/evidence/ftp_version_s1_output.txt`  
**Follow-ups:** run `nmap --script ftp-syst,ftp-anon -p21 <target>`.

---

### ftp_login — Brute-force / credential testing
**Module**
```
auxiliary/scanner/ftp/ftp_login
```

**Important options**
- `USER_FILE`, `PASS_FILE`, `USERPASS_FILE`, `STOP_ON_SUCCESS`, `BRUTEFORCE_SPEED`  
**Typical output (snippet)**  
```
[+] 192.228.115.3:21 - Login Successful: sysadmin:654321
```  
**Evidence:**  
`service-enumeration/ftp/evidence/ftp_login_s2_output.txt` (save command + wordlist names)  
**Caution:** brute-force may lock accounts / trigger IDS.

---

### anonymous — Test anonymous login
**Module**
```
auxiliary/scanner/ftp/anonymous
```

**What it does:** attempts `anonymous` login and may list directories.  
**Typical snippet**  
```
[+] demo.ine.local:21 - anonymous login allowed (read-only)
```  
**If write allowed:** manually test `put` only in lab and save proof to evidence.  
**Evidence:** `service-enumeration/ftp/evidence/ftp_anonymous_s3_output.txt`

---

### ftp (CLI) — Manual interaction
**Tool**
```
ftp <target>
# interactive commands (ls, get, put, cd)
```

**Record session log to:** `service-enumeration/ftp/evidence/ftp_cli_session_<date>.txt`  
**Example session**
```
Name (demo.ine.local:user): sysadmin
230 User sysadmin logged in
ftp> ls
ftp> get config.php
ftp> quit
```

---

## ✅ Final checklist (FTP enumeration)
- [ ] Save raw module output and command history to `service-enumeration/ftp/evidence/`.  
- [ ] Store screenshots in `service-enumeration/ftp/screenshots/`.  
- [ ] Note valid credentials only in local evidence (never commit).  
- [ ] Create `service-enumeration/ftp/ftp-cheatsheet.md` (1-page) with key commands.  
- [ ] Export flashcards CSV: `service-enumeration/ftp/ftp-flashcards.csv`.  
- [ ] Add follow-ups to `service-enumeration/ftp/followups.md` (convert to GitHub issues).
