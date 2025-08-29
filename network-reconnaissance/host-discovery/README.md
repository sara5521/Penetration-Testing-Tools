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
### 🔹 2. fping (Range Ping)
```bash
fping -a -g <target-ip>
```
- -a: Show only a live hosts
- -g: IP range
### 🔹 3. Nmap Ping Scan
```bash
nmap -sn <target-ip>
```
### 🔹 4. ARP Scan (Local Network)
```bash
arp-scan <target-ip>
```
### 🔹 5. Netdiscover (Interactive)
```bash
netdiscover -r <target-ip>
```
### 🔹 6. Masscan (Caution!)
```bash
masscan <target-ip> -p0-65535 --rate=1000
```
