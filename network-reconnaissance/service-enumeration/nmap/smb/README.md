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

### 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```

### 🔄 4. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```

### 📂 5. Enumerate and Browse SMB Shares

- List shared folders:
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```
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

---

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts may require authentication.
- Combine with tools like:
  - ```enum4linux```
  - ```smbclient```
  - ```crackmapexec```
