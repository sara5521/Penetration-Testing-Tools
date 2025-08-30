# 📦 Nmap - SMB Enumeration Scripts

This section includes useful Nmap scripts to enumerate SMB (Server Message Block) services on Windows machines.

---

## 🎯 Purpose

These scripts help gather important information like:
- OS version
- Hostname & domain
- Shared folders (shares)
- User accounts
- SMB protocol version
- Security settings (e.g., SMB signing)

---

## 🔧 Common SMB Nmap Scripts

### 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```

### 👥 2. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```

### 📂 3. Enumerate Shares
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```
### 🔐 4. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```

### 🔄 5. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```

### 🌐 6. Detect SMB Protocol Version
```bash
nmap -p 445 --script smb-protocols <target-ip>
```

### 🔁 All-in-One Scan
```bash
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode" <target-ip>
```

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts may require authentication.
- Combine with tools like:
  - enum4linux
  - smbclient
  - crackmapexec
