# 🛠️ Penetration Testing Tools Collection

This repository is a structured collection of penetration testing tools organized by phase and category.  
Each folder includes practical commands, usage examples, and tips for common tools used in ethical hacking and CTFs.

---

## 🧭 Why This Repo?

- To help you study and practice tools used in eJPT, TryHackMe, Hack The Box, etc.
- To keep all commands, notes, and cheat sheets organized in one place.
- To build a personal knowledge base for real-world assessments and labs.

---

## 📂 Folder Structure

```
Penetration-Testing-Tools/
│
├── network-reconnaissance/     # Basic network info, host discovery, port scanning
│   ├── basics/
│   ├── host-discovery/
│   ├── port-scanning/
│
├── service-enumeration/        # Banner grabbing, SMB/FTP/HTTP enumeration, etc.
│   ├── nmap/
│   ├── smb/
│   ├── ftp/
│   └── http/
│
├── vulnerability-scanning/     # Tools to find known vulnerabilities
│   ├── nikto/
│   ├── nessus/
│   └── nmap-vuln/
│
├── password-attacks/           # Brute force and password cracking tools
│   ├── hydra/
│   ├── medusa/
│   └── john-the-ripper/
│
├── web-pentesting/             # Directory fuzzing, Burp Suite, Wfuzz, etc.
│   ├── dirb-gobuster/
│   ├── burpsuite/
│   └── wfuzz/
│
├── post-exploitation/          # Tools used after gaining access (e.g., privilege escalation)
│   ├── mimikatz/
│   └── privilege-escalation/
│
├── misc/                       # Cheatsheets, methodology, extra notes
│   ├── cheatsheets/
│   └── methodology/
```

---

## 🔗 Tools Covered (Examples)

- **Nmap** - scanning, scripts, enumeration
- **Enum4linux** - SMB enumeration
- **Hydra & Medusa** - brute-force
- **Nikto** - web vuln scanner
- **Burp Suite** - web app testing
- **Wfuzz, Gobuster** - directory & parameter fuzzing
- **John the Ripper** - password cracking

---

## 👩‍💻 Author

Created by `sara5521`  
Cybersecurity student & eJPT candidate

---

## ✅ Status

🟢 Actively updating this repo with INE labs, CTF notes, and tool cheatsheets.  
📅 Goal: Complete before INE subscription ends.

---

## 📌 License

MIT License — feel free to copy, fork, or use for learning.
