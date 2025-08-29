# 🚪 Nmap - Port Scanning & Service Enumeration

Nmap is one of the most powerful tools used for **port scanning** and **service detection** in penetration testing.

---

## 🔧 Basic Port Scanning

- Scan default 1000 ports:
  ```bash
  nmap <target-ip>
  ```
- Scan specific port:
  ```bash
  nmap -p 80 <target-ip>
  ```
- Scan multiple ports:
  ```bash
  nmap -p 21,22,80 <target-ip>
  ```
- Scan all 65535 ports:
  ```bash
  nmap -p- <target-ip>
  ```

 ## 🔍 Service & Version Detection
  
- Detect running services and versions:
  ```bash
  nmap -sV <target-ip>
  ```
- Aggressive scan (OS, services, scripts, traceroute):
  ```bash
  nmap -A <target-ip>
  ```

## 🎯 Common Scan Combinations

- Fast + detailed:
  ```bash
  nmap -T4 -sV -p- <target-ip>
  ```
- Top 100 ports:
  ```bash
  nmap --top-ports 100 <target-ip>
  ```
- Save results:
  ```bash
  nmap -oN results.txt <target-ip>
  ```

## 🧠 Tips
- Use -T4 for faster scans (aggressive timing).
- Use -Pn if ping is blocked:
  ```bash
  nmap -Pn <target-ip>
  ```
- Combine with NSE scripts for deeper enumeration (e.g., --script smb-*, --script http-*).

