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

## 📜 NSE Scripts for Enhanced Enumeration

NSE (Nmap Scripting Engine) provides powerful scripts for deeper service enumeration:

### HTTP Service Enumeration
```bash
# Basic HTTP enumeration - finds directories like /webdav
nmap --script http-enum -sV -p 80 demo.ine.local

# Find directories, headers, and HTTP methods
nmap --script http-enum,http-headers,http-methods -p 80,443 <target>

# Check for common web vulnerabilities
nmap --script http-vuln-* -p 80,443 <target>
```

**Example Output from WebDAV Lab:**
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-enum: 
|   /webdav/: Potentially interesting folder (401 Unauthorized)
|_  /admin/: Potentially interesting folder
|_http-server-header: Microsoft-IIS/10.0
```

### SMB Enumeration
```bash
# SMB share enumeration
nmap --script smb-enum-shares -p 139,445 <target>

# SMB vulnerabilities (EternalBlue, etc.)
nmap --script smb-vuln-* -p 139,445 <target>
```

### Other Common Service Scripts
```bash
# FTP anonymous login check
nmap --script ftp-anon -p 21 <target>

# FTP bounce attack check
nmap --script ftp-bounce -p 21 <target>

# SSH enumeration
nmap --script ssh2-enum-algos -p 22 <target>

# DNS zone transfer
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<domain> -p 53 <target>
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
- Comprehensive scan with scripts:
  ```bash
  nmap -sS -sV -sC -O -p- <target-ip>
  ```
- Quick initial assessment:
  ```bash
  nmap -sS -sV -F <target-ip>
  ```

---

## 💾 Output Formats & Saving Results

You can save Nmap scan results in various formats for reporting, analysis, or automation.

### 📝 Output Format Options

| Format | Flag | Use Case | Description |
|--------|------|----------|-------------|
| **Normal** | `-oN file.txt` | Human readable | Standard text output |
| **XML** | `-oX file.xml` | Automation/parsing | Structured data format |
| **Grepable** | `-oG file.gnmap` | Text processing | Grep-friendly format |
| **All formats** | `-oA basename` | Complete documentation | Creates .nmap, .xml, .gnmap files |

### Examples:
```bash
# Save in normal format
nmap -sV -oN scan_results.txt demo.ine.local

# Save in XML format (good for tools integration)
nmap -sV -Pn -oX myscan.xml demo.ine.local

# Save in all formats at once
nmap -sS -sV -sC -oA comprehensive_scan demo.ine.local
```

### 🔎 Working with Results
You can later process results with:
- `xsltproc` (convert XML to HTML)
- Python `xml.etree.ElementTree`
- `Nmap Parser` (Python modules)
- Import into vulnerability scanners like OpenVAS
- Parse with grep/awk for specific information

---

## 🚀 Timing and Performance Options

Control scan speed and stealth with timing templates:

| Template | Flag | Speed | Detection Risk | Use Case |
|----------|------|-------|----------------|----------|
| **Paranoid** | `-T0` | Very Slow | Very Low | Maximum stealth, avoids IDS |
| **Sneaky** | `-T1` | Slow | Low | Avoid detection, careful enumeration |
| **Polite** | `-T2` | Moderate | Medium | Normal enumeration, avoid overwhelming target |
| **Normal** | `-T3` | Normal | Medium | Default setting, general scanning |
| **Aggressive** | `-T4` | Fast | High | Lab/CTF environments, time-sensitive tests |
| **Insane** | `-T5` | Very Fast | Very High | Speed over accuracy, may miss results |

### Examples:
```bash
# Fast scan for lab environments
nmap -T4 -sS -sV -p- <target>

# Stealth scan for production systems
nmap -T1 -sS -p 80,443 <target>
```

---

## 🧪 Real Lab Examples

Practical examples from penetration testing scenarios:

### Example 1: Initial Network Discovery
```bash
# Discover live hosts in network
nmap -sn 192.168.1.0/24

# Quick port scan on discovered host  
nmap -T4 -F demo.ine.local
```

### Example 2: WebDAV Lab Enumeration (Step by Step)
```bash
# Step 1: Basic port scan - discovers open ports
nmap demo.ine.local

# Step 2: Service version detection - identifies IIS and versions
nmap -sV demo.ine.local

# Step 3: HTTP-specific enumeration - finds /webdav directory
nmap --script http-enum -sV -p 80 demo.ine.local
```

**Expected Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-10 09:36 IST
Nmap scan report for demo.ine.local (10.0.31.40)
Host is up (0.00027s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3306/tcp open  mysql         MySQL (unauthorized)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

### Example 3: Full Network Assessment
```bash
# Discover live hosts
nmap -sn 192.168.1.0/24

# Quick port scan
nmap -sS -T4 -F <target-ip>

# Comprehensive scan with all information
nmap -sS -sV -sC -O -p- -T4 -oA full_assessment <target-ip>

# Focus on web services
nmap --script http-* -p 80,443,8080,8443 <target-ip>
```

---

## 🧠 Tips & Best Practices

### For eJPT Exam:
- Always use `-sS` when possible (requires root)
- Combine `-sV` for service detection
- Use `-T4` for faster scanning in lab environments
- Save results with `-oA` for complete documentation
- Focus on HTTP enumeration with `--script http-enum`
- Start with quick scans (`-F` or `--top-ports 1000`) then expand to full port scans (`-p-`)

### General Tips:
- Use `-Pn` if ping is blocked or filtered:
  ```bash
  nmap -Pn <target-ip>
  ```
- Always save scan results for documentation:
  ```bash
  nmap -sS -sV -oA initial_scan <target>
  ```
- Combine basic scanning with NSE scripts for thorough enumeration
- Document everything with proper output formatting

### Common Port Ranges:
```bash
# Top 100 most common ports
nmap --top-ports 100 <target>

# Top 1000 ports (default)
nmap --top-ports 1000 <target>

# Common web ports
nmap -p 80,443,8080,8443 <target>

# Common Windows ports
nmap -p 135,139,445,3389 <target>

# Common services
nmap -p 21,22,23,25,53,80,110,143,443,993,995 <target>
```

---

## 🎯 Quick Reference Commands

### Essential Nmap Commands for eJPT:
```bash
# Basic enumeration
nmap -sS -sV -sC <target>

# HTTP service discovery
nmap --script http-enum -p 80,443 <target>

# Full port scan with comprehensive output
nmap -sS -sV -p- -oA scan_results <target>

# SMB enumeration
nmap --script smb-enum-shares -p 139,445 <target>

# Fast initial assessment
nmap -sS -T4 -F <target>

# Network discovery
nmap -sn <network-range>

# Common services scan
nmap -sS -sV --top-ports 1000 <target>
```

### Host Discovery Options:
```bash
# Ping scan only (no port scan)
nmap -sn <network>

# Skip host discovery (assume host is up)
nmap -Pn <target>

# TCP SYN discovery on specific ports
nmap -PS80,443 <network>

# UDP discovery
nmap -PU53 <network>
```

---

## 🎯 eJPT Exam Focus Areas

Key Nmap techniques for eJPT certification:

1. **Basic Port Scanning**: `-sS`, `-sV`, `-p-`, `--top-ports`
2. **Service Enumeration**: `--script http-enum`, `--script smb-*`
3. **Output Management**: `-oA` for complete documentation  
4. **Timing**: `-T4` for efficient scanning in lab environments
5. **Host Discovery**: `-sn` for network mapping, `-Pn` when ping blocked
6. **NSE Scripts**: Focus on HTTP, SMB, and FTP enumeration scripts
