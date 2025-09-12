# 📦 Metasploit - SMB Version Enumeration

This tool helps identify the **exact version** of the SMB (Samba) server running on the target.

---

## 📂 Path in GitHub Project

```
service-enumeration/smb/metasploit.md
```

---

## 🎯 Purpose

The `smb_version` auxiliary module in Metasploit helps you:
- Identify the exact version of the Samba/SMB service
- Detect supported SMB dialects (e.g., SMBv1, SMBv2, SMBv3)
- Gather OS details of the target (e.g., "Samba 4.3.11-Ubuntu")

---

## 🛠️ Module Used

```
auxiliary/scanner/smb/smb_version
```

---

## 💡 How to Use It

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS <target-ip or hostname>
exploit
```

✅ **Example (INE Lab)**

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS demo.ine.local
exploit
```

---

## 📸 Sample Output

```bash
[*] 192.220.69.3:445 - SMB Detected (versions: 1, 2, 3)
[*] Host is running: Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] Auxiliary module execution completed
```

---

## 🔍 Interpretation

- ✅ **Samba 4.3.11-Ubuntu** → This is the exact version of the Samba service.
- 🪪 Knowing the version helps you search for specific vulnerabilities (e.g. CVEs).
- ⚠️ Older versions may be vulnerable to exploits like EternalBlue or RCE.

---

## 🧠 Pro Tips

- Use this module before trying SMB exploits (like EternalBlue).
- Combine it with `nmap -p445 --script smb-*` for full enumeration.
- Try using it on different hosts in the network to compare SMB versions.

---

## 🔗 Related Modules

| Module Path                             | Purpose                        |
|----------------------------------------|--------------------------------|
| `auxiliary/scanner/smb/smb_version`    | Detect SMB version             |
| `auxiliary/scanner/smb/smb_enumshares` | List SMB shares (if enabled)   |
| `auxiliary/scanner/smb/smb_login`      | Brute-force SMB credentials    |

---

📚 [Metasploit Docs](https://docs.rapid7.com/metasploit/)
