# Metasploit - FTP Enumeration Modules (Detailed)

## Purpose
Metasploit auxiliary modules to enumerate FTP services, test anonymous login, and perform credential checks.

---

## Table of contents
- ftp_version — detect server and version  
- ftp_login — credential brute-force / credential testing  
- anonymous — anonymous access checks  
- ftp (manual) — interactive client usage and post-login actions

---

## ftp_version — Detect FTP server and version

**Module**
```
auxiliary/scanner/ftp/ftp_version
```

**Purpose (short)**
Grab FTP banner and extra version info. Quick reconnaissance to target follow-up checks.

**Important options**
- `RHOSTS` — target host or CIDR.  
- `RPORT` (default 21) — target port.  
- `THREADS` — parallel scans. Use lower value in labs to be polite.

**What it does**
- Connects to TCP port (default 21).  
- Reads server greeting/banner.  
- Attempts simple FTP SYST or identification commands if banner is incomplete.

**Typical output**
```
[+] 192.0.2.3:21 - FTP Server: ProFTPD 1.3.5a
```

**Interpretation**
- Banner shows service and version. Use this to map known CVEs and exploitability. Banner can be forged or stripped. Do not assume full accuracy.

**False positives / caveats**
- Admins may change banner text.
- Some FTP servers hide version. If banner missing, try `ftp-syst` script in nmap or manual `SYST`.

**Follow-ups**
- Run targeted nmap scripts (ftp-syst, ftp-anon).  
- Search CVE and vendor advisories for the exact version.  
- If version vulnerable, test in a controlled lab only.

**Evidence to collect**
- Raw banner text, timestamp, command used. Save XML or console log.

**Defence / remediation notes (for reports)**
- Hide or truncate product/version in banner.  
- Patch the FTP daemon. Disable unneeded FTP if not required. Use SFTP or FTPS where possible.

---

## ftp_login — Brute-force / credential testing

**Module**
```
auxiliary/scanner/ftp/ftp_login
```

**Purpose (short)**
Attempt login using credential lists. Confirm weak passwords and valid accounts.

**Important options**
- `RHOSTS` — target(s).  
- `USER_FILE` — file of usernames.  
- `PASS_FILE` — file of passwords.  
- `USERPASS_FILE` — file with `user:pass` pairs (optional).  
- `BRUTEFORCE_SPEED` / `THREADS` — control rate to reduce account lockouts.  
- `STOP_ON_SUCCESS` — stop after first valid credential found.

**What it does**
- Iterates through usernames and passwords or uses paired list.  
- Attempts anonymous or null login if configured.  
- Reports successful logins.

**Typical output**
```
[+] 192.228.115.3:21 - Login Successful: sysadmin:654321
```

**Interpretation**
- A valid credential was found. This grants immediate access level of that account. Use it to enumerate files and permissions manually.

**Operational cautions**
- Brute force can trigger IDS/IPS or lock accounts.  
- Use slow rate and permissioned lab only.  
- Respect legal/ethical limits.

**Tuning tips**
- Prioritize targeted username lists (from OSINT or enumerations).  
- Use `STOP_ON_SUCCESS` to avoid excess noise.  
- Test a small sample first.

**Follow-ups**
- Use found credentials with `ftp` or `smbclient` depending on services.  
- Search for reused credentials on other services.  
- Try to escalate or pivot only in lab scenarios.

**Evidence to collect**
- Exact command, wordlist names, timestamps, and successful lines in raw output.

**Defence / remediation notes**
- Enforce account lockout policies.  
- Enforce strong password policies.  
- Monitor failed login spikes and block offending IPs. Use MFA where possible.

---

## anonymous — Test anonymous login

**Module**
```
auxiliary/scanner/ftp/anonymous
```

**Purpose (short)**
Check if server permits anonymous or guest access. Reports read/write permissions where possible.

**Important options**
- `RHOSTS` — target.  
- `RPORT` — port (21).  
- `ANON_USER` / `ANON_PASS` — rarely needed; defaults are common.

**What it does**
- Connects and attempts `anonymous` or `ftp` login.  
- Lists root directory when allowed.  
- Checks for write permission by attempting simple operations when module supports it (some versions only list).

**Typical output**
```
[+] demo.ine.local:21 - scanned 1 of 1 hosts (100% complete)
[+] Auxiliary module execution completed
```
You may also see file listings or messages indicating write allowed.

**Interpretation**
- Anonymous allowed means attacker can list and possibly upload files depending on permissions. This is a misconfiguration risk.

**Common caveats**
- Some servers allow read-only anonymous; others allow upload. Module may not always attempt uploads to avoid destructive actions.

**Follow-ups**
- If anonymous allowed, manually connect and test `ls`, `get`, and cautiously `put` if lab rules allow. Verify write permission explicitly.  
- Search uploaded files for sensitive data or web shells if web root writable.

**Evidence to collect**
- Directory listing, presence of unexpected files, timestamps, and upload tests (if performed).

**Defence / remediation notes**
- Disable anonymous login if not required.  
- Restrict anonymous to read-only and sandboxed directories.  
- Audit uploaded content and logs regularly.

---

## ftp (CLI) — Manual interaction after credentials

**Tool**
```
ftp <target>
# or use nc/telnet for banner interaction
```

**Purpose (short)**
Interactive session to list files, download, upload, and inspect remote FS.

**When to use**
- After finding credentials or confirming anonymous access.  
- For manual validation and evidence gathering.

**Common session flow**
```
Connected to demo.ine.local.
220 (ProFTPD 1.3.5a)
Name (demo.ine.local:user): sysadmin
331 Please specify the password.
230 User sysadmin logged in
ftp> ls
ftp> get file.txt
ftp> put test.txt    # only if write allowed and permitted
ftp> quit
```

**Useful client commands**
- `ls` / `dir` — list files.  
- `cd <dir>` — change directory.  
- `get <file>` — download.  
- `put <file>` — upload.  
- `mget` / `mput` — multiple files.  
- `binary` / `ascii` — transfer mode.

**Operational cautions**
- Avoid uploading destructive content in labs unless required.  
- Preserve timestamps and paths for reporting. Use `-v` or logging to capture raw session.

**Follow-ups**
- Use downloaded files to search credentials or configuration.  
- If writable directories map to web root, check for remote code execution via uploaded web shell.

**Defence / remediation notes**
- Limit user home directories, enforce upload quarantine, scan uploaded files.  
- Remove anonymous upload and use secure transport (SFTP/FTPS).

---

## Final checklist for FTP enumeration (use after module runs)
- [ ] Save raw module output and command history.  
- [ ] Record timestamps and RHOSTS list.  
- [ ] Note any valid credentials and where they were found.  
- [ ] Check for writable anonymous directories.  
- [ ] Map findings to follow-up actions: download, search, escalate, or report.  
- [ ] Include remediation guidance in report.
