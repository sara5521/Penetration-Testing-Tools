# 📦 Nmap - SMB Enumeration Scripts

This section includes useful Nmap scripts to enumerate SMB (Server Message Block) services on Windows machines.

---

## 🎯 Purpose

These scripts help you gather important information from the SMB service, such as:
- Operating system and hostname
- Shared folders (shares)
- Users and groups
- Domain or workgroup names
- Security configurations
- Server statistics and uptime
- Protocol versions
- Running services

---

## 🔧 Common SMB Nmap Scripts

### 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```

### 🌐 2. Detect SMB Protocol Version
```bash
nmap -p 445 --script smb-protocols <target-ip>
```
#### 📌 Purpose:
This script shows which SMB version the server supports:
- SMBv1 → Old and not secure ⚠️
- SMBv2 / SMBv3 → Newer and more secure ✅
#### 🧠 Why is this useful?
- If the server supports SMBv1, it's risky and may be vulnerable to attacks like EternalBlue.
- Tells you whether modern, secure protocols (SMBv3) are supported.
- Important for deciding what kind of attack or enumeration you can try.

### 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```
#### 📌 Purpose:
This script checks how secure the SMB server is by showing:
- If SMB signing is supported
- If SMB signing is required
- What type of authentication is used (user or share level)
#### 🧠 Why it's useful?
- If signing is not required, the server might be vulnerable to spoofing or MiTM (man-in-the-middle) attacks.
- Helps you understand how the target is protected.

### 🔄 4. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```
#### 📌 Purpose:
This script tries to list current SMB sessions — in other words, it shows:
- Who is currently connected to the SMB server
- What users or machines are active
- How many open sessions exist
#### 🧠 Why is this useful?
- Shows real-time activity on the target.
- Helps identify:
  - Logged-in users
  - IPs of other attackers or admins
  - If the system is actively being used

### 📂 5. Enumerate and Browse SMB Shares

- List shared folders:
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```
#### 📌 Purpose:
This script lists all shared folders (also called shares) on a Windows machine over SMB.
It tells you:
- Which folders are being shared
- Whether they are readable or writable
- If they're administrative or public shares

#### 🧠 Why is this useful?
- Helps you find files or folders that are publicly accessible
- Great for finding sensitive data like ```flag.txt```, ```backup.zip```, or ```config.bak```.
- If a share is writable, you might be able to upload malicious files.

- List files inside the shares:
```bash
nmap -p 445 --script smb-ls <target-ip>
```
Use ```smb-enum-shares``` to identify available shares, and ```smb-ls``` to see the contents inside them.

### 👥 6. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```

### 📊 7. View SMB Server Stats
```bash
nmap -p 445 --script smb-server-stats <target-ip>
```
Displays:
- Open files
- Active sessions
- Logged in users
- Uptime (if available)

### 🏢 8. Enumerate SMB Domains / Workgroups
```bash
nmap -p 445 --script smb-enum-domains <target-ip>
```
- Attempts to list Windows domains or workgroups.
- Useful in internal AD environments.

### 👤 9. Enumerate User Groups
```bash
nmap -p 445 --script smb-enum-groups <target-ip>
```
- Lists local or domain user groups on the target.
- Helps in identifying roles like Administrators, Guests, Remote Users.

### ⚙️ 10. Enumerate Running Services
```bash
nmap -p 445 --script smb-enum-services <target-ip>
```
- Lists Windows services running on the target via SMB.
- Helps identify exposed or vulnerable services.
  
### 🔁 All-in-One Scan
```bash
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats" <target-ip>
```

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts may require authentication.
- Combine with tools like:
  - ```enum4linux```
  - ```smbclient```
  - ```crackmapexec```
