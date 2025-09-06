# 🌐 Network Reconnaissance

This phase is the foundation of any penetration test. It focuses on discovering live hosts, identifying open ports, and preparing for deeper enumeration.

---

## 🛠️ Tools & Techniques Covered

| Category         | Tools & Commands                       |
|------------------|----------------------------------------|
| **Basic Concepts** | Port fundamentals, TCP vs UDP, use cases |
| **Host Discovery** | `ping`, `nmap -sn`, `nmap -Pn`, `arp-scan` |
| **Port Scanning** | `nmap -sS`, `nmap -sT`, `nmap -sU`, `nmap -p`, `nmap -A` |

---

## 🎯 Purpose

Before diving into deep enumeration, this stage helps you:

- Identify live hosts on the target network
- Discover open and filtered ports
- Understand basic networking principles (IP, ports, protocols)
- Prepare for targeted scanning (e.g., SMB, FTP, HTTP)
- Map the attack surface of the target

---

## 📁 Folder Navigation

| Folder             | Description                                                 |
|--------------------|-------------------------------------------------------------|
| `basics/`          | Networking fundamentals such as ports, TCP vs UDP, etc.     |
| `host-discovery/`  | Tools for detecting live systems and bypassing firewalls    |
| `port-scanning/`   | Techniques to scan and enumerate open ports using Nmap      |

---

## 🚀 Quick Examples

```bash
# Host discovery (ping scan)
nmap -sn 192.168.1.0/24

# Stealth TCP SYN port scan
nmap -sS -p- 192.168.1.10

# Detect OS and services
nmap -A 192.168.1.10
```
