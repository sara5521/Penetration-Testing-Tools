# 🔎 Service Enumeration

This section focuses on enumerating **specific services** (like SMB, FTP, HTTP, SMTP...) running on open ports after discovering them via port scanning.

Each service has its own subfolder containing useful tools, Nmap scripts, and enumeration methods.

---

## 📁 Services Covered

| Service | Tools & Techniques |
|--------|---------------------|
| [SMB](./smb/) | `nmap`, `enum4linux` |
| [FTP](./ftp/) | `nmap`, `hydra` |
| [HTTP](./http/) | `nikto`, `gobuster`, `nmap` |
| [SMTP](./smtp/) | `nmap`, `smtp-user-enum` |

---

## 🎯 Purpose

After identifying open ports (e.g., 21, 80, 445, 25...), this stage helps you:

- Detect service versions
- Identify misconfigurations
- Find hidden files/folders or shares
- Extract users, banners, directories
- Build deeper knowledge of the target system

---

## 🧭 Navigation

| Folder | Description |
|--------|-------------|
| [`smb/`](./smb/) | Server Message Block enumeration: users, shares, domains, stats |
| [`ftp/`](./ftp/) | FTP service checks: anonymous login, brute-force, version |
| [`http/`](./http/) | Web server scanning, hidden files/folders, tech fingerprinting |
| [`smtp/`](./smtp/) | Email-related service enumeration: usernames, relay testing |

---

## 📌 Tip

If you're scanning a target and you see one of these ports open, jump to the relevant folder:

| Port | Service |
|------|---------|
| 445  | SMB     |
| 21   | FTP     |
| 80/443 | HTTP |
| 25   | SMTP    |

---

## 📚 Related Topics

- [Network Reconnaissance](../network-reconnaissance/)
- [Vulnerability Scanning](../vulnerability-scanning/)
- [Post Exploitation](../post-exploitation/)
