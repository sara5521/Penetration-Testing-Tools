# 🌐 Host Discovery - Live Host Scanning

This section lists the most common tools and commands used to discover **live hosts** in a network — a critical first step in reconnaissance and penetration testing.

---

## 📊 Tools Comparison

| Tool            | Description                                             | Notes                              |
|-----------------|---------------------------------------------------------|-------------------------------------|
| `ping`          | Sends ICMP Echo to check if host is alive               | May be blocked by firewalls         |
| `fping`         | Sends ping to multiple IPs in a range                   | Faster than `ping`, supports ranges |
| `nmap -sn`      | Performs a Ping Scan to list only online hosts          | Great for networks (e.g., /24)      |
| `arp-scan`      | Scans the local network using ARP                       | Layer 2 scan — accurate on LAN      |
| `netdiscover`   | ARP-based live host discovery with interactive output   | Easy to use in local networks       |
| `masscan`       | Ultra-fast scanner for IP/port discovery                | Can overwhelm networks — use carefully |

---

## 🧪 Example Commands

### 🔹 1. Ping (Basic)
```bash
ping <target-ip>
```
### 🔹 1. Ping (Basic)
```bash
ping <target-ip>
```
