# 🔌 Understanding Network Ports

This file explains what a **port** is and lists some of the most common ports used in networking and penetration testing.

---

## 📦 What is a Port?

A **port** is a logical access point for network services on a computer.  
Each device has an IP address, and each service on the device runs on a specific port.

Think of it like this:
- IP address = Building address
- Port number = Apartment number
Each service (web server, FTP, email, etc.) listens on its own port.

---

## 🏢 Real-Life Analogy

A **computer** is like an apartment building.  
The **IP address** is the street address.  
Each **port** is a different room or apartment where a specific service lives.

---

## 📋 Common Ports and Their Uses

| Port | Service / Protocol | Description                     |
|------|---------------------|---------------------------------|
| 21   | FTP                 | File Transfer Protocol          |
| 22   | SSH                 | Secure Shell (remote login)     |
| 23   | Telnet              | Unencrypted remote login (legacy) |
| 25   | SMTP                | Email sending                   |
| 53   | DNS                 | Domain Name System              |
| 80   | HTTP                | Web (insecure)                  |
| 110  | POP3                | Email receiving                 |
| 143  | IMAP                | Email access                    |
| 443  | HTTPS               | Secure web traffic              |
| 445  | SMB                 | Windows file sharing            |
| 3306 | MySQL               | MySQL Database service          |
| 3389 | RDP                 | Remote Desktop Protocol         |

---

## 🎯 Why is this important in Pentesting?

- Ports reveal what services are available on a target.
- Some ports may expose **vulnerabilities**.
- Tools like `nmap` can help discover **open ports** and what services are running.

---

## 🧪 Nmap Example

```bash
nmap -p 21,22,80,445 <target-ip>
```

This scans the FTP, SSH, HTTP, and SMB ports on the target.

---

## 🧠 Tip

Use this information to decide:
- What enumeration scripts or exploits to run
- Whether a port/service needs authentication or allows anonymous access
- Which tools to use next (`nmap`, `hydra`, `smbclient`, etc.)
