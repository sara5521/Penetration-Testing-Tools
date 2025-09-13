# 📡 Metasploit: SMB Version Enumeration

This module is used to detect the version of SMB running on a target system using Metasploit.

---

## 🎯 Purpose

To identify which version of the SMB (Server Message Block) protocol is running on the target host. This helps in:
- Identifying vulnerabilities (e.g., EternalBlue affects SMBv1)
- Planning further attacks or enumeration steps

---

## 🛠️ Metasploit Module

**Module Path:**
```
scanner/smb/smb_version
```

---

## 📌 Usage Steps

### 1. Start Metasploit Console
```bash
msfconsole
```

### 2. Use the SMB Version Scanner Module
```bash
use scanner/smb/smb_version
```

### 3. Set the Target IP
```bash
set RHOSTS <target-ip>
```
Replace `<target-ip>` with the IP address of the target machine.

### 4. Run the Scan
```bash
run
```

---

## 📸 Example
```bash
msf6 > use scanner/smb/smb_version
msf6 auxiliary(scanner/smb/smb_version) > set RHOSTS 10.10.45.13
RHOSTS => 10.10.45.13
msf6 auxiliary(scanner/smb/smb_version) > run

[*] 10.10.45.13:445       - SMB Detected (versions:) (preferred dialect: SMB 2.1)
[*] 10.10.45.13:445       -   OS: Windows 6.1 (build: 7601) Service Pack 1
```

---

## 🧠 Notes

- If the module returns `SMB 1.0`, that could be a sign of an outdated system.
- Windows 6.1 → Windows 7 or Windows Server 2008 R2
- This module is useful for OS fingerprinting and vulnerability checking.

---

## ✅ Related Tools

- `nmap --script smb-protocols` → Also shows supported SMB versions
- `smbclient` → Connect to SMB shares after discovering version

---

## 🧠 Tips

- If Metasploit throws a timeout, make sure port 445 is open on the target.
- You can use a range for RHOSTS: `set RHOSTS 10.10.45.1-254`
- Combine with `smb_enumshares`, `smb_login`, or `smb_enumusers` for deeper enumeration.
