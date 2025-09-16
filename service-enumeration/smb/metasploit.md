# Metasploit - SMB Version Enumeration (Detailed)

## Purpose
Detailed notes for Metasploit auxiliary modules used to enumerate SMB (Samba) services.  
Use this file for fingerprinting SMB versions, detecting supported SMB dialects, enumerating shares, and testing authentication.

---

## Key modules / usage
- `auxiliary/scanner/smb/smb_version` — detect SMB/Samba version and OS details.  
- `auxiliary/scanner/smb/smb_enumshares` — list SMB shares (anonymous or authenticated).  
- `auxiliary/scanner/smb/smb_login` — brute-force SMB credentials.  
- `auxiliary/scanner/smb/smb_enumusers` — enumerate user accounts (when available).  
- `auxiliary/scanner/smb/smb_enumgroups` — enumerate groups (when available).  

---

## Quick examples
```bash
# Start msfconsole
msfconsole -q

# Detect SMB version
use auxiliary/scanner/smb/smb_version
set RHOSTS <target>
run

# Enumerate shares (anonymous)
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target>
run

# Brute-force login
use auxiliary/scanner/smb/smb_login
set RHOSTS <target>
set USER_FILE /usr/share/wordlists/users.txt
set PASS_FILE /usr/share/wordlists/passwords.txt
run
```

---

## How to read common output
- `SMB Detected (versions: 1, 2, 3)` → indicates supported SMB dialects.  
- `Samba 4.3.11-Ubuntu` → banner showing product and version — use to check CVEs.  
- `\HOST\share FOUND` or `READ, WRITE` → accessible share and permissions.  
- `Login Successful: user:pass` → valid credentials discovered.  
- `Anonymous login allowed` → anonymous access to IPC$/shares may be possible.

---

## Practical tips
- Run `smb_version` early to determine if SMBv1 is enabled (legacy risk).  
- Combine with Nmap NSE scripts (`smb-os-discovery`, `smb-enum-shares`, `smb-vuln-*`) for cross-validation.  
- When brute-forcing, use `STOP_ON_SUCCESS true` to save time.  
- Respect lab rules — do aggressive or destructive checks only in authorized environments.  
- Save raw output and timestamps (`spool` in msfconsole) for reporting and reproducibility.  
- If valid creds found, use `smbclient`, `rpcclient`, or mount the share to explore contents safely.

---

# Module details (expanded)

### smb_version — Detect SMB/Samba version
**Module**
```
auxiliary/scanner/smb/smb_version
```

**Purpose (short)**  
Profile SMB/Samba banner, dialects and OS details.

**Important options**
- `RHOSTS` — target host(s)  
- `RPORT` — default 445  
- `THREADS` — parallelism

**What it does**
- Performs SMB negotiation and inspects banner/response to infer product/version and supported dialects.

**Typical output**
```
[*] 192.220.69.3:445 - SMB Detected (versions: 1, 2, 3)
[*] Host is running: Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] Auxiliary module execution completed
```

**Interpretation**
- Exact Samba version → map to vendor advisories and CVEs.  
- Presence of SMBv1 indicates higher risk (legacy protocols like EternalBlue).

**False positives / caveats**
- Banners can be obfuscated by admins or proxied by SMB gateways.  
- Some systems hide version information; cross-check with other tools.

**Follow-ups**
- Run Nmap NSE `smb-os-discovery`, `smb-enum-shares`, and vulnerability scripts.  
- Search CVE databases for the exact version.

**Evidence to collect**
- Raw msfconsole output, timestamps, and any banner strings.

**Remediation**
- Patch Samba/SMB, disable SMBv1, and follow vendor guidance.

---

### smb_enumshares — Enumerate SMB shares
**Module**
```
auxiliary/scanner/smb/smb_enumshares
```

**Purpose (short)**  
List shares exposed by the SMB server (anonymous or authenticated).

**Important options**
- `RHOSTS`, `SMBUser` / `SMBPass` (if using credentials)

**What it does**
- Connects to SMB and lists available shares and access rights.

**Typical output**
```
[+] \192.220.69.3\IPC$ - Remote IPC
[+] \192.220.69.3\public - READ, WRITE
```

**Interpretation**
- `public` with WRITE access can be used to upload files in lab environments.  
- IPC$ is normal; other writable shares are higher-value targets.

**Caveats**
- Enumeration results may vary depending on anon access and ACLs.

**Follow-ups**
- Use `smbclient //host/share -N` or `smbclient -U user` to list files.  
- Download and analyze any interesting files in an isolated environment.

**Remediation**
- Restrict share permissions, remove sensitive files from shares, implement access controls.

---

### smb_login — Brute-force SMB login
**Module**
```
auxiliary/scanner/smb/smb_login
```

**Purpose (short)**  
Attempt authentication using credential lists.

**Important options**
- `USER_FILE`, `PASS_FILE`, `USERPASS_FILE`, `STOP_ON_SUCCESS`  
- `RHOSTS`, `THREADS`

**What it does**
- Submits credential combinations and reports successful logins.

**Typical output**
```
[+] 192.220.69.3:445 - Login Successful: admin:password123
```

**Interpretation**
- Valid creds can be used to access shares, RDP, or other services depending on environment.

**Caveats**
- Brute force can lock accounts or trigger detections — use carefully in labs.

**Follow-ups**
- After success, enumerate shares and attempt to pivot using valid credentials.

**Remediation**
- Enforce strong passwords, account lockout policies, and MFA where possible.

---

### smb_enumusers / smb_enumgroups — Enumerate users & groups
**Modules**
```
auxiliary/scanner/smb/smb_enumusers
auxiliary/scanner/smb/smb_enumgroups
```

**Purpose (short)**  
Enumerate user and group information exposed via SMB (when supported).

**What it does**
- Attempts to list users/groups via RPC/SMB queries.

**Typical output**
```
[+] Found user: alice
[+] Found group: Domain Admins
```

**Interpretation**
- Usernames are useful for targeted brute-force or social engineering.

**Caveats**
- Not all SMB setups expose user lists; results may be limited.

**Follow-ups**
- Use discovered usernames with `smb_login` or other services.

**Remediation**
- Limit exposure of user/group info and apply least privilege.

---

## INE Lab Example (complete scenario)

### Input
```bash
# Detect SMB version
use auxiliary/scanner/smb/smb_version
set RHOSTS demo.ine.local
run

# Enumerate shares anonymously
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS demo.ine.local
run

# Brute-force SMB login (if allowed)
use auxiliary/scanner/smb/smb_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/wordlists/users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
run
```

### Sample Output
```
[*] 192.220.69.3:445 - SMB Detected (versions: 1, 2, 3)
[*] Host is running: Windows 6.1 (Samba 4.3.11-Ubuntu)
[+] \192.220.69.3\public - READ, WRITE
[+] 192.220.69.3:445 - Login Successful: admin:password123
```

### Short analysis and next steps
- Target runs **Samba 4.3.11-Ubuntu** → check CVEs and vendor advisories.  
- SMBv1 supported → treat as high-risk; consider MS17-010/EternalBlue checks in lab.  
- Writable share `public` → may allow file uploads (test only in lab).  
- Valid credentials → use to enumerate further and pivot.

---

## Final checklist (report)
- [ ] Save msfconsole raw output logs and timestamps.  
- [ ] Note exact modules and options used.  
- [ ] Cross-check Samba/SMB version against CVE databases.  
- [ ] Enumerate and safely inspect shares (hash and store findings).  
- [ ] Provide remediation steps (patching, disable SMBv1, ACLs, password policy).  

---

[Metasploit Docs](https://docs.rapid7.com/metasploit/)
