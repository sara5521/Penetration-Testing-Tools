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

## 🗂️ SMB Enumeration Scripts Summary

| 🔢 | Script                  | What it Does                             |
|----|-------------------------|-------------------------------------------|
| 1️⃣ | smb-os-discovery       | Detect OS and hostname                   |
| 2️⃣ | smb-protocols          | Show SMB versions supported              |
| 3️⃣ | smb-security-mode      | Check SMB signing and auth level         |
| 4️⃣ | smb-enum-domains       | List domain/workgroup info               |
| 5️⃣ | smb-enum-users         | List user accounts                       |
| 6️⃣ | smb-enum-groups        | List Windows groups                      |
| 7️⃣ | smb-enum-sessions      | List active SMB sessions                 |
| 8️⃣ | smb-enum-services      | List running services                    |
| 9️⃣ | smb-enum-shares        | List shared folders                      |
| 🔟 | smb-ls                  | List files inside shares                 |
| 1️⃣1️⃣ | smb-server-stats     | Show stats like open files/sessions      |
| 1️⃣2️⃣ | --script-args        | Customize script input (username, etc.)  |

---

## 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```

---

## 🌐 2. Detect SMB Protocol Version
```bash
nmap -p 445 --script smb-protocols <target-ip>
```
### 📌 Purpose:
This script shows which SMB version the server supports:
- SMBv1 → Old and not secure ⚠️
- SMBv2 / SMBv3 → Newer and more secure ✅

### 🧠 Why is this useful?
- If the server supports SMBv1, it's risky and may be vulnerable to attacks like EternalBlue.
- Helps you decide what kind of attacks/enumeration you can try.

---

## 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```
### 📌 Purpose:
Checks how secure the SMB server is (signing, auth level).

### 🧠 Why is this useful?
- If signing is not required, server may be vulnerable to spoofing or MiTM.
- Helps assess target protection.

---

## 🏢 4. Enumerate SMB Domains / Workgroups
```bash
nmap -p 445 --script smb-enum-domains <target-ip>
```

---

## 👥 5. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```

---

## 👤 6. Enumerate User Groups
```bash
nmap -p 445 --script smb-enum-groups <target-ip>
```

---

## 🔄 7. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```

---

## ⚙️ 8. Enumerate Running Services
```bash
nmap -p 445 --script smb-enum-services <target-ip>
```

---

## 📂 9. Enumerate and Browse SMB Shares

### List shared folders:
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```

### List files inside the shares:
```bash
nmap -p 445 --script smb-ls <target-ip>
```

---

## 📊 10. View SMB Server Stats
```bash
nmap -p 445 --script smb-server-stats <target-ip>
```

---

## 🧾 11. Using `--script-args` in Nmap
```bash
nmap -p 445 --script <script-name> --script-args <key>=<value>
```

### 🧪 Examples:
- With login:
```bash
nmap -p 445 --script smb-enum-users --script-args smbuser=admin,smbpass=123 <target-ip>
```
- With domain:
```bash
nmap -p 445 --script smb-enum-shares --script-args smbuser=user,smbpass=123,domain=WORKGROUP <target-ip>
```
- With timeout:
```bash
nmap -p 445 --script smb-enum-sessions --script-args timeout=20s <target-ip>
```

### 🔧 Common script-args:
| Key       | Description                 |
|-----------|-----------------------------|
| `smbuser` | Username for SMB login      |
| `smbpass` | Password for SMB login      |
| `domain`  | Windows domain or workgroup |
| `timeout` | Script timeout (e.g. 10s)   |

---

## 🔁 All-in-One Scan
```bash
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats" <target-ip>
```

---

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts need authentication.
- Combine with tools like:
  - `enum4linux`
  - `smbclient`
  - `crackmapexec`
