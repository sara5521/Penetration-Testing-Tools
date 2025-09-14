# 🚪 Nmap - Port Scanning & Service Enumeration

Nmap is one of the most powerful tools used for **port scanning** and **service detection** in penetration testing.

---

## 🔎 Basic Scan Types

- **TCP Connect Scan (-sT)**  
  - Completes the full 3-way handshake.  
  - Works without root privileges.  
  - Easier to detect by firewalls/IDS.  

- **SYN Scan (-sS)**  
  - Half-open scan (does not complete the handshake).  
  - Faster and stealthier.  
  - Requires root privileges.  

- **UDP Scan (-sU)**  
  - Scans UDP ports.  
  - Slower than TCP scans.  
  - Useful for services like DNS (53), SNMP (161), DHCP (67/68).  

---

### 📊 Comparison Table

| Scan Type | Flag | Pros | Cons |
|-----------|------|------|------|
| **TCP Connect** | `-sT` | Works without root, reliable results | Slow, easy to detect by IDS/IPS |
| **SYN Scan** | `-sS` | Fast, stealthy, less detectable | Needs root privileges |
| **UDP Scan** | `-sU` | Finds UDP services (DNS, SNMP, DHCP) | Very slow, many false negatives, often blocked by firewalls |

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

---

## 🔍 Service & Version Detection

- Detect running services and versions:
  ```bash
  nmap -sV <target-ip>
  ```
- Aggressive scan (OS, services, scripts, traceroute):
  ```bash
  nmap -A <target-ip>
  ```

---

## ⚔️ SYN Scan (-sS) vs Version Detection (-sV)

### 📋 Difference Between Commands

| Command | Feature | Description |
|---------|---------|-------------|
| `nmap -sV <target-ip>` | **Service Version Detection** | Scans open ports and tries to detect the version of the service. |
| `nmap -sS -sV <target-ip>` | **SYN Scan + Version Detection** | Adds SYN scan (half-open TCP scan) with version detection. Faster and stealthier. |

---

### ✅ Switch Details

**`-sV`**
- Detects services running on open ports.  
- Shows versions like `Apache 2.4.49`, `OpenSSH 7.6`, etc.  
- Useful for finding possible vulnerabilities.  

**`-sS`**
- Uses **SYN scan** (*half-open scan*).  
- Does not complete the **three-way handshake**:  
  - If reply is **SYN-ACK** → Port is open.  
  - If reply is **RST** → Port is closed.  
- Faster and stealthier than a full connect scan.  
- Requires root privileges.

---

### 🧠 When to Use

| Situation | Best Command |
|-----------|--------------|
| Just need to know service versions | `nmap -sV <target-ip>` |
| Need fast and stealthy scan with versions | `nmap -sS -sV <target-ip>` |
| Sensitive target or lab testing | Always use `-sS` with other options |

---

### 🧪 Examples

#### 1️⃣ Service Version Detection Only
```bash
nmap -sV <target-ip>
```
- Shows open services and versions.  
- Uses the default scan type:  
  - Root user → SYN scan.  
  - Non-root user → TCP connect scan.

#### 2️⃣ SYN Scan + Version Detection
```bash
nmap -sS -sV <target-ip>
```
- Same results for services and versions.  
- But runs **SYN scan**, which is faster and stealthier.

---

### 📈 Visual Comparison: TCP Connect vs SYN Scan

#### 🔗 TCP Connect Scan (-sT)
```
Client                      Server
   | ----- SYN -----------> |
   | <---- SYN/ACK -------- |
   | ----- ACK -----------> |   ✅ Full 3-way handshake
   |                        |
   | ----- RST -----------> |   ❌ Close connection
```
- Completes the handshake.  
- Easier to detect by IDS/IPS.  
- Used if you run Nmap without root.

#### ⚡ SYN Scan (-sS)
```
Client                      Server
   | ----- SYN -----------> |
   | <---- SYN/ACK -------- |
   | ----- RST -----------> |   ⭕ Half-open connection
```
- Sends SYN, receives SYN-ACK, then resets.  
- Does **not** complete the handshake.  
- Faster and more stealthy.  
- Requires root privileges.

---

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

---

## 💾 Saving Nmap Results

You can save Nmap scan results in various formats for reporting, analysis, or automation.

### 📝 Save in XML format
```bash
nmap -sV -Pn -oX myscan.xml demo.ine.local
```
- `-sV`: Detects service versions  
- `-Pn`: Skips host discovery (assumes host is up)  
- `-oX`: Output in XML format  
- `myscan.xml`: Output file name  

🔎 You can later open this XML with tools like:
- `xsltproc` (to convert to HTML)
- `Nmap Parser` (Python modules)
- Import into vulnerability scanners

---

## 🧠 Tips
- Use `-T4` for faster scans (aggressive timing).  
- Use `-Pn` if ping is blocked:
  ```bash
  nmap -Pn <target-ip>
  ```
- Combine with NSE scripts for deeper enumeration (e.g., `--script smb-*`, `--script http-*`).  
