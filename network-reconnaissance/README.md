# 🌐 Network Reconnaissance

This folder contains tools and commands used during the **initial phase of penetration testing**, which focuses on discovering live hosts and open ports.

---

## 📂 Folder Structure

```
network-reconnaissance/
│
├── basics/
│   └── Ports_Basics.md
│
├── host-discovery/
│   └── README.md
│
└── port-scanning/
    ├── port-basics.md
    └── README.md
```

---

## 🧱 1. basics/Ports_Basics.md

Explains the fundamentals of networking ports:
- What are ports?
- TCP vs UDP
- Common port numbers and their uses
- Examples:
  - 21 → FTP
  - 22 → SSH
  - 80 → HTTP
  - 443 → HTTPS
  - 445 → SMB

---

## 📡 2. host-discovery/README.md

Identifying live hosts on the network using tools like `ping`, `nmap`, and `netdiscover`.

### 🔍 Example Commands:

```bash
# ICMP Ping
ping 192.168.1.1

# ARP Discovery (LAN)
netdiscover -r 192.168.1.0/24

# Ping Sweep using Nmap
nmap -sn 192.168.1.0/24
```

---

## 🚪 3. port-scanning/README.md

Scanning for open ports to discover running services.

### 🔍 Example Nmap Scans:

```bash
# TCP SYN Scan (Stealth)
nmap -sS <target-ip>

# Full TCP Connect
nmap -sT <target-ip>

# UDP Port Scan
nmap -sU <target-ip>

# Version Detection
nmap -sV <target-ip>

# Disable Ping (treat host as online)
nmap -Pn <target-ip>

# Export scan results
nmap -oX scan.xml <target-ip>
```

📘 `port-basics.md` explains:
- Open / Closed / Filtered ports
- TCP 3-Way Handshake
- Nmap scan types

---

## 🧠 Tips

- Use `-Pn` if ICMP is blocked (to avoid host discovery).
- Always follow up with service enumeration after port scanning.
- Consider exporting results for use with other tools.

---

Happy Scanning! 🎯
