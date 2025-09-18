# 🚪 Comprehensive Nmap Guide - Port Scanning & Service Enumeration

Nmap is one of the most powerful tools used for **port scanning** and **service detection** in penetration testing and network security assessments.

---

## 🔎 Basic Scan Types

### Core Scanning Methods

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

### 📊 Scan Type Comparison Table

| Scan Type | Flag | Pros | Cons |
|-----------|------|------|------|
| **TCP Connect** | `-sT` | Works without root, reliable results | Slow, easy to detect by IDS/IPS |
| **SYN Scan** | `-sS` | Fast, stealthy, less detectable | Needs root privileges |
| **UDP Scan** | `-sU` | Finds UDP services (DNS, SNMP, DHCP) | Very slow, many false negatives, often blocked by firewalls |

---

## 🔧 Basic Port Scanning Commands

### Essential Port Scanning
```bash
# Scan default 1000 ports
nmap <target-ip>

# Scan specific port
nmap -p 80 <target-ip>

# Scan multiple ports
nmap -p 21,22,80 <target-ip>

# Scan all 65535 ports
nmap -p- <target-ip>

# Scan top ports
nmap --top-ports 100 <target-ip>
nmap --top-ports 1000 <target-ip>
```

### Host Discovery
```bash
# Ping scan only (discover live hosts)
nmap -sn <network-range>

# Skip host discovery (assume host is up)
nmap -Pn <target>

# TCP SYN discovery on specific ports
nmap -PS80,443 <network>

# UDP discovery
nmap -PU53 <network>
```

---

## 🔍 Service & Version Detection

### Version Detection Commands
```bash
# Detect running services and versions
nmap -sV <target-ip>

# Aggressive scan (OS, services, scripts, traceroute)
nmap -A <target-ip>

# OS detection
nmap -O <target-ip>

# Service detection with faster timing
nmap -sV -T4 <target-ip>
```

### ⚔️ SYN Scan (-sS) vs Version Detection (-sV)

#### 📋 Command Differences

| Command | Feature | Description |
|---------|---------|-------------|
| `nmap -sV <target-ip>` | **Service Version Detection** | Scans open ports and tries to detect the version of the service. |
| `nmap -sS -sV <target-ip>` | **SYN Scan + Version Detection** | Adds SYN scan (half-open TCP scan) with version detection. Faster and stealthier. |

#### ✅ Switch Details

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

#### 📈 Visual Comparison: TCP Connect vs SYN Scan

**🔗 TCP Connect Scan (-sT)**
```
Client                      Server
   | ----- SYN -----------> |
   | <---- SYN/ACK -------- |
   | ----- ACK -----------> |   ✅ Full 3-way handshake
   |                        |
   | ----- RST -----------> |   ❌ Close connection
```

**⚡ SYN Scan (-sS)**
```
Client                      Server
   | ----- SYN -----------> |
   | <---- SYN/ACK -------- |
   | ----- RST -----------> |   ⭕ Half-open connection
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

# HTTP title and server information
nmap --script http-title,http-server-header -p 80,443 <target>

# All HTTP scripts
nmap --script http-* -p 80,443 <target>
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

# SMB OS discovery
nmap --script smb-os-discovery -p 139,445 <target>

# All SMB scripts
nmap --script smb-* -p 139,445 <target>
```

### Specialized Web Service Discovery

#### WebDAV Discovery and Enumeration

**WebDAV (Web Distributed Authoring and Versioning) Testing with Nmap**

**Basic WebDAV Discovery**
```bash
# Discover WebDAV-enabled directories
nmap --script http-enum -sV -p 80,443 <target>

# Specific WebDAV scanning
nmap --script http-webdav-scan -p 80,443 <target>

# HTTP methods enumeration
nmap --script http-methods --script-args http-methods.url-path=/webdav/ <target>

# Complete WebDAV assessment
nmap --script http-webdav-scan,http-methods,http-enum -p 80,443 <target>
```

**Real Lab Example: Windows IIS WebDAV Discovery**

**Scenario:** Discovering WebDAV service on Windows IIS Server
- **Target:** demo.ine.local (10.0.31.40)
- **Service:** Microsoft IIS httpd 10.0
- **Objective:** Identify WebDAV capabilities and directory structure

**Step 1: Initial Port Scan**
```bash
nmap demo.ine.local
```

**Expected Lab Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-10 09:36 IST
Nmap scan report for demo.ine.local (10.0.31.40)
Host is up (0.0027s latency).
Not shown: 990 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 1.44 seconds
```

**Step 2: HTTP Service Enumeration**
```bash
nmap --script http-enum -sV -p 80 demo.ine.local
```

**Expected Lab Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-10 09:37 IST
Nmap scan report for demo.ine.local (10.0.31.40)
Host is up (0.0026s latency).

PORT    STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-enum:
|_  /webdav/: Potentially interesting folder (401 Unauthorized)
|_http-server-header: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.57 seconds
```

**Analysis:** 
- **WebDAV Directory Found:** `/webdav/` directory discovered with 401 Unauthorized status
- **Authentication Required:** HTTP Basic authentication needed for access
- **Server Platform:** Microsoft IIS 10.0 on Windows
- **Potential Vulnerability:** WebDAV service may allow file upload if credentials obtained

**Step 3: WebDAV-Specific Scanning**
```bash
nmap --script http-webdav-scan -p 80 demo.ine.local
```

**Expected Results:**
- Confirms WebDAV service availability
- Identifies supported HTTP methods (PUT, MOVE, COPY, DELETE)
- Reveals authentication requirements

**Additional WebDAV Detection Scripts**
```bash
# Check for common WebDAV paths with detailed enumeration
nmap --script http-enum --script-args http-enum.displayall -p 80 <target>

# Test HTTP methods on specific paths
nmap --script http-methods --script-args http-methods.url-path=/webdav/,http-methods.test-all -p 80 <target>

# Comprehensive WebDAV and HTTP enumeration
nmap --script http-webdav-scan,http-methods,http-enum,http-title,http-server-header -p 80,443 <target>
```

**Common WebDAV Paths to Test**
```bash
# Test multiple potential WebDAV endpoints
nmap --script http-enum --script-args http-enum.basepath=/webdav/ -p 80 <target>
nmap --script http-enum --script-args http-enum.basepath=/dav/ -p 80 <target>
nmap --script http-enum --script-args http-enum.basepath=/uploads/ -p 80 <target>
nmap --script http-enum --script-args http-enum.basepath=/_vti_bin/ -p 80 <target>
```

**Common WebDAV Locations:**
- `/webdav/` - Standard WebDAV directory
- `/dav/` - Alternative WebDAV path
- `/uploads/` - Upload directory with WebDAV
- `/_vti_bin/` - Microsoft FrontPage WebDAV
- `/exchange/` - Exchange WebDAV
- `/public/` - Public WebDAV access

**eJPT Integration Points**

**WebDAV Discovery in eJPT Context:**
1. **Service Enumeration:** Use nmap to discover WebDAV services
2. **Authentication Testing:** Identify authentication requirements
3. **Directory Discovery:** Find WebDAV-enabled paths
4. **Follow-up Testing:** Use davtest and cadaver for exploitation

**Critical Commands for eJPT:**
```bash
# Primary discovery
nmap --script http-enum -sV -p 80 <target>

# WebDAV confirmation  
nmap --script http-webdav-scan -p 80 <target>

# Methods enumeration
nmap --script http-methods --script-args http-methods.url-path=/webdav/ <target>

# Comprehensive WebDAV assessment
nmap --script http-webdav-scan,http-methods,http-enum -p 80,443 <target>
```

**Integration with Other Tools:**
- **Discovery:** nmap → identifies WebDAV service
- **Assessment:** davtest → tests upload capabilities  
- **Exploitation:** cadaver → manual file upload and management
- **Access:** web browser → web shell execution

**Complete WebDAV Assessment Workflow:**
```bash
# 1. Initial discovery
nmap --script http-enum -sV -p 80,443 <target>

# 2. WebDAV confirmation  
nmap --script http-webdav-scan -p 80,443 <target>

# 3. Method testing
nmap --script http-methods --script-args http-methods.url-path=/webdav/ <target>

# 4. Follow-up with specialized tools
davtest -url http://<target>/webdav/
cadaver http://<target>/webdav/
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

# SMTP enumeration
nmap --script smtp-enum-users -p 25 <target>

# MySQL enumeration
nmap --script mysql-info -p 3306 <target>
```

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

### Timing Examples
```bash
# Fast scan for lab environments
nmap -T4 -sS -sV -p- <target>

# Stealth scan for production systems
nmap -T1 -sS -p 80,443 <target>

# Balanced approach
nmap -T3 -sS -sV --top-ports 1000 <target>
```

---

## 💾 Output Formats & Saving Results

Save Nmap scan results in various formats for reporting, analysis, or automation.

### 📝 Output Format Options

| Format | Flag | Use Case | Description |
|--------|------|----------|-------------|
| **Normal** | `-oN file.txt` | Human readable | Standard text output |
| **XML** | `-oX file.xml` | Automation/parsing | Structured data format |
| **Grepable** | `-oG file.gnmap` | Text processing | Grep-friendly format |
| **All formats** | `-oA basename` | Complete documentation | Creates .nmap, .xml, .gnmap files |

### Output Examples
```bash
# Save in normal format
nmap -sV -oN scan_results.txt demo.ine.local

# Save in XML format (good for tools integration)
nmap -sV -Pn -oX myscan.xml demo.ine.local

# Save in all formats at once
nmap -sS -sV -sC -oA comprehensive_scan demo.ine.local

# Append results to existing file
nmap -sS -sV --append-output -oN existing_scan.txt <target>
```

### 🔎 Working with Results
You can later process results with:
- `xsltproc` (convert XML to HTML)
- Python `xml.etree.ElementTree`
- `Nmap Parser` (Python modules)
- Import into vulnerability scanners like OpenVAS
- Parse with grep/awk for specific information

---

## 🎯 Common Scan Combinations

### Quick Assessment Scans
```bash
# Fast initial assessment
nmap -sS -T4 -F <target-ip>

# Quick comprehensive scan
nmap -sS -sV -sC -T4 <target-ip>

# Top 100 ports
nmap --top-ports 100 <target-ip>
```

### Comprehensive Scans
```bash
# Full port scan with all information
nmap -sS -sV -sC -O -p- <target-ip>

# Aggressive comprehensive scan
nmap -A -T4 -p- <target-ip>

# Script-heavy enumeration
nmap -sS -sV -sC --script vuln -p- <target-ip>
```

### Targeted Service Scans
```bash
# Web services focus
nmap -sS -sV --script http-* -p 80,443,8080,8443 <target>

# Windows services focus
nmap -sS -sV --script smb-*,ms-* -p 135,139,445,3389 <target>

# Database services
nmap -sS -sV -p 1433,3306,5432,1521 <target>
```

---

## 🧪 Real Lab Examples

Practical examples from penetration testing scenarios:

### Example 1: Network Discovery Phase
```bash
# Step 1: Discover live hosts in network
nmap -sn 192.168.1.0/24

# Step 2: Quick port scan on discovered hosts
nmap -T4 -F 192.168.1.10

# Step 3: Detailed scan on interesting hosts
nmap -sS -sV -sC -T4 192.168.1.10
```

### Example 2: WebDAV Lab Enumeration (Complete Workflow)
```bash
# Step 1: Basic port scan - discovers open ports
nmap demo.ine.local

# Step 2: Service version detection - identifies IIS and versions
nmap -sV demo.ine.local

# Step 3: HTTP-specific enumeration - finds /webdav directory
nmap --script http-enum -sV -p 80 demo.ine.local

# Step 4: Full HTTP enumeration
nmap --script http-* -p 80 demo.ine.local
```

**Expected Output:**
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-10 09:36 IST
Nmap scan report for demo.ine.local (10.0.31.40)
Host is up (0.00027s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-enum: 
|   /webdav/: Potentially interesting folder (401 Unauthorized)
|_  /admin/: Potentially interesting folder
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3306/tcp open  mysql         MySQL (unauthorized)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

### Example 3: Full Network Assessment Workflow
```bash
# Phase 1: Discovery
nmap -sn 192.168.1.0/24

# Phase 2: Quick enumeration of live hosts
nmap -sS -T4 --top-ports 1000 <target-list>

# Phase 3: Comprehensive scan of interesting hosts
nmap -sS -sV -sC -O -p- -T4 -oA full_assessment <target-ip>

# Phase 4: Service-specific enumeration
nmap --script http-*,smb-*,ftp-* -p 21,80,139,443,445 <target-ip>
```

---

## 🎯 Common Port Ranges & Services

### Standard Port Ranges
```bash
# Web services
nmap -p 80,443,8080,8443,8000,8888 <target>

# Common Windows ports
nmap -p 135,139,445,3389,5985,5986 <target>

# Database services
nmap -p 1433,3306,5432,1521,27017 <target>

# Mail services
nmap -p 25,110,143,993,995,587 <target>

# DNS and network services
nmap -p 53,67,68,161,162 <target>

# Remote access services
nmap -p 22,23,3389,5900,5901 <target>
```

### Service-Specific Scans
```bash
# All common services
nmap -p 21,22,23,25,53,80,110,135,139,143,443,445,993,995,3306,3389,5432 <target>

# Quick web assessment
nmap -sS -sV --script http-title,http-server-header -p 80,443 <target>

# SMB assessment
nmap -sS -sV --script smb-os-discovery,smb-enum-shares -p 139,445 <target>
```

---

## 🧠 Advanced Techniques & Tips

### Firewall and IDS Evasion
```bash
# Fragment packets
nmap -f <target>

# Use decoy IPs
nmap -D RND:10 <target>

# Source port specification
nmap --source-port 53 <target>

# Random host order
nmap --randomize-hosts <target-list>

# Slow timing to avoid detection
nmap -T1 -sS <target>
```

### Advanced NSE Usage
```bash
# Run all safe scripts
nmap -sC <target>

# Run specific script categories
nmap --script auth,safe <target>

# Run scripts with arguments
nmap --script http-enum --script-args http-enum.basepath=/admin/ <target>

# Custom script execution
nmap --script /usr/share/nmap/scripts/custom-script.nse <target>
```

### Performance Optimization
```bash
# Parallel host scanning
nmap --min-hostgroup 50 <network>

# Parallel port scanning
nmap --min-parallelism 100 <target>

# Skip DNS resolution
nmap -n <target>

# Increase send rate
nmap --min-rate 1000 <target>
```

---

## 🎯 eJPT Exam Focus Areas

### Essential Commands for eJPT Certification
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

### Key Areas to Master:

1. **Basic Port Scanning**: `-sS`, `-sV`, `-p-`, `--top-ports`
2. **Service Enumeration**: `--script http-enum`, `--script smb-*`
3. **Output Management**: `-oA` for complete documentation  
4. **Timing**: `-T4` for efficient scanning in lab environments
5. **Host Discovery**: `-sn` for network mapping, `-Pn` when ping blocked
6. **NSE Scripts**: Focus on HTTP, SMB, and FTP enumeration scripts

### Best Practices for eJPT:

- Always use `-sS` when possible (requires root)
- Combine `-sV` for service detection
- Use `-T4` for faster scanning in lab environments
- Save results with `-oA` for complete documentation
- Focus on HTTP enumeration with `--script http-enum`
- Start with quick scans (`-F` or `--top-ports 1000`) then expand to full port scans (`-p-`)
- Document everything with proper output formatting

---

## 🚨 Common Issues & Troubleshooting

### When Scans Don't Work
```bash
# If ping is blocked
nmap -Pn <target>

# If ports seem closed but services are running
nmap -sT <target>  # Try TCP connect instead of SYN

# For slow networks
nmap -T2 --max-retries 3 <target>

# If getting permission denied
sudo nmap -sS <target>  # Use sudo for SYN scans
```

### Performance Issues
```bash
# Speed up scans
nmap -T4 --min-rate 1000 <target>

# Reduce resource usage
nmap -T2 --max-parallelism 10 <target>

# Skip unnecessary features
nmap -n --disable-arp-ping <target>
```

---

## 📚 Quick Reference Card

### Most Used Commands
```bash
# Discovery
nmap -sn <network>

# Basic scan
nmap -sS -sV <target>

# Full scan
nmap -sS -sV -sC -p- <target>

# Web enumeration
nmap --script http-enum -p 80,443 <target>

# Save everything
nmap -sS -sV -sC -oA results <target>

# Fast assessment
nmap -T4 -F <target>
```

### Essential Flags
- `-sS` = SYN scan (stealth)
- `-sV` = Version detection
- `-sC` = Default scripts
- `-p-` = All ports
- `-T4` = Fast timing
- `-oA` = Save all formats
- `-Pn` = Skip ping
- `--script` = Run NSE scripts

This comprehensive guide covers all essential Nmap techniques needed for penetration testing, network security assessments, and certification exams like eJPT.
