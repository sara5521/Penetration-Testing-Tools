# 📁 FTP Service Enumeration with Nmap

Complete guide for FTP (File Transfer Protocol) service discovery and enumeration using Nmap NSE scripts.

---

## 📋 Table of Contents
- [Purpose and Overview](#purpose-and-overview)
- [Script Reference Table](#script-reference-table)
- [Prerequisites](#prerequisites)
- [Service Detection and Version Scanning](#service-detection-and-version-scanning)
- [Anonymous Access Testing](#anonymous-access-testing)
- [System Information Gathering](#system-information-gathering)
- [Brute Force Testing](#brute-force-testing)
- [Vulnerability Detection](#vulnerability-detection)
- [Advanced Techniques](#advanced-techniques)
- [Complete Assessment Workflow](#complete-assessment-workflow)
- [Real Lab Examples](#real-lab-examples)
- [Output Analysis and Interpretation](#output-analysis-and-interpretation)
- [Integration with Other Tools](#integration-with-other-tools)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [eJPT Focus Points](#ejpt-focus-points)

---

## Purpose and Overview

Nmap FTP enumeration helps you:
- Detect FTP service versions and configurations
- Test for anonymous FTP access and file permissions
- Gather system information from FTP servers
- Perform credential brute-force attacks safely
- Identify FTP-specific vulnerabilities and backdoors
- Enumerate FTP server capabilities and features

---

## Script Reference Table

| Script | Purpose | Auth Required | Risk Level | CVE Detection |
|--------|---------|---------------|------------|---------------|
| `ftp-anon` | Test anonymous access | No | Medium | - |
| `ftp-syst` | System information | No | Low | - |
| `ftp-brute` | Credential brute-force | No | High | - |
| `ftp-vsftpd-backdoor` | vsFTPd backdoor detection | No | Medium | CVE-2011-2523 |
| `ftp-bounce` | FTP bounce attack testing | No | High | CVE-1999-0017 |
| `ftp-proftpd-backdoor` | ProFTPd backdoor detection | No | Medium | Various |

---

## Prerequisites

### Installation and Setup
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install nmap

# CentOS/RHEL
sudo yum install nmap

# Verify FTP scripts availability
nmap --script-help ftp-*
locate ftp-anon.nse
```

### Quick Service Verification
```bash
# Basic port scan
nmap -p 21 <target>

# Quick service detection
nmap -sV -p 21 <target>
```

---

## Service Detection and Version Scanning

### Basic Service Detection
```bash
# Standard service version detection
nmap -sV -p 21 <target>
```

**Example Output:**
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
```

### Enhanced Service Detection with Scripts
```bash
# Service detection with default scripts
nmap -sV -sC -p 21 <target>
```

**Enhanced Output:**
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.100
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

### Comprehensive Service Analysis
```bash
# Aggressive service detection
nmap -sV -A -p 21 <target>

# Service detection with all FTP scripts
nmap -sV --script "ftp-* and not ftp-brute" -p 21 <target>
```

### Common FTP Server Identification

#### vsFTPd Detection
```bash
21/tcp open  ftp     vsftpd 3.0.3
```
- **Software**: Very Secure FTP Daemon (Linux)
- **Security**: Generally secure implementation
- **Vulnerability**: Check version 2.3.4 for backdoor

#### ProFTPd Detection
```bash
21/tcp open  ftp     ProFTPD 1.3.5a
```
- **Software**: Professional FTP daemon
- **Security**: History of security issues
- **Vulnerability**: Various CVEs, especially older versions

#### FileZilla Server
```bash
21/tcp open  ftp     FileZilla ftpd 0.9.60
```
- **Software**: FileZilla Server (Windows)
- **Platform**: Windows-based FTP server
- **Security**: Check for latest updates

---

## Anonymous Access Testing

### Basic Anonymous Access Detection
```bash
# Test for anonymous FTP access
nmap --script ftp-anon -p 21 <target>
```

**Example Output:**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_List of FTP files and folders
```

### Detailed Anonymous Enumeration
```bash
# Anonymous access with file listing limit
nmap --script ftp-anon --script-args ftp-anon.maxlist=10 -p 21 <target>

# Deep directory traversal
nmap --script ftp-anon --script-args ftp-anon.maxlist=100,ftp-anon.maxdepth=2 -p 21 <target>

# Check specific FTP directories
nmap --script ftp-anon --script-args ftp-anon.basedir="/pub/" -p 21 <target>
```

**Detailed Output Analysis:**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 0        0            4096 Sep 16 12:00 uploads [NSE: writeable]
| drwxr-xr-x    2 0        0            4096 Sep 16 12:00 documents  
| -rw-r--r--    1 0        0             156 Sep 16 12:00 readme.txt
| -rw-r--r--    1 0        0            2048 Sep 16 12:00 config.bak
|_End of listing
```

### Anonymous Access Script Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `maxlist` | Maximum files to list | `--script-args ftp-anon.maxlist=20` |
| `maxdepth` | Directory traversal depth | `--script-args ftp-anon.maxdepth=2` |
| `basedir` | Starting directory | `--script-args ftp-anon.basedir="/pub/"` |

### Permission Analysis
| Permission String | Meaning | Security Risk |
|------------------|---------|---------------|
| `drwxrwxrwx` | Fully writable directory | Critical - File upload possible |
| `drwxr-xr-x` | Read-only directory | Medium - Information disclosure |
| `-rw-r--r--` | Read-only file | Low - Limited access |
| `-rw-rw-rw-` | Writable file | High - File modification possible |

---

## System Information Gathering

### FTP System Information
```bash
# Gather FTP server system information
nmap --script ftp-syst -p 21 <target>
```

**Example Output:**
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.100
|      Logged in as ftp
|      TYPE: ASCII
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

### Combined System and Access Testing
```bash
# Comprehensive system information with anonymous testing
nmap --script ftp-syst,ftp-anon -sV -p 21 <target>

# System information with banner grabbing
nmap --script banner,ftp-syst -p 21 <target>
```

---

## Brute Force Testing

### Basic Brute Force Authentication
```bash
# Basic credential brute force (USE WITH CAUTION)
nmap --script ftp-brute -p 21 <target>
```

**Warning**: Brute force attacks can lock accounts and trigger security alerts.

### Advanced Brute Force with Custom Wordlists
```bash
# Custom username and password lists
nmap --script ftp-brute --script-args userdb=users.txt,passdb=passwords.txt -p 21 <target>

# Safe brute force with limitations
nmap --script ftp-brute --script-args brute.threads=2,unpwdb.timelimit=30,brute.firstonly=true -p 21 <target>

# Brute force with common credentials only
nmap --script ftp-brute --script-args brute.firstonly=true -p 21 <target>
```

**Example Output:**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-brute: 
|   Accounts: 
|     admin:password123 - Valid credentials
|     ftp:ftp - Valid credentials
|   Statistics: Performed 25 guesses in 15 seconds
|_  Average tps: 1.7
```

### Brute Force Script Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `userdb` | Username list file | `userdb=/path/to/users.txt` |
| `passdb` | Password list file | `passdb=/path/to/passwords.txt` |
| `unpwdb.timelimit` | Time limit in seconds | `unpwdb.timelimit=30` |
| `brute.firstonly` | Stop after first success | `brute.firstonly=true` |
| `brute.threads` | Number of threads | `brute.threads=3` |
| `brute.delay` | Delay between attempts | `brute.delay=1s` |

---

## Vulnerability Detection

### vsFTPd Backdoor Detection (CVE-2011-2523)
```bash
# Detect vsFTPd 2.3.4 backdoor
nmap --script ftp-vsftpd-backdoor -p 21 <target>
```

**Example Output (if vulnerable):**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-vsftpd-backdoor: 
|   VULNERABLE:
|   vsFTPd version 2.3.4 backdoor
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2011-2523
|     Risk factor: High  CVSSv2: 10.0 (HIGH) (AV:N/AC:L/Au:N/C:C/I:C/A:C)
|       vsFTPd version 2.3.4 backdoor, this was reported on 2011-07-04.
|     Disclosure date: 2011-07-03
|     References:
|       http://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-2523
|_      http://www.securityfocus.com/bid/48539
```

### ProFTPd Backdoor Detection
```bash
# Detect ProFTPd backdoors
nmap --script ftp-proftpd-backdoor -p 21 <target>
```

### FTP Bounce Attack Testing (CVE-1999-0017)
```bash
# Test for FTP bounce attack vulnerability
nmap --script ftp-bounce -p 21 <target>

# FTP bounce with specific internal target
nmap --script ftp-bounce --script-args ftp-bounce.checkhost=192.168.1.10 -p 21 <target>
```

**Example Output (if vulnerable):**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-bounce: 
|   VULNERABLE:
|   FTP bounce attack
|     State: VULNERABLE
|     IDs:  CVE:CVE-1999-0017
|       FTP bounce attack allows an attacker to use the FTP server
|       to port scan other hosts or to connect to arbitrary ports on other machines.
|_    References: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0017
```

### Comprehensive Vulnerability Assessment
```bash
# All vulnerability checks
nmap --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce -p 21 <target>
```

---

## Advanced Techniques

### Comprehensive FTP Assessment
```bash
# All FTP scripts except brute force
nmap --script "ftp-* and not ftp-brute" -sV -p 21 <target>

# Complete FTP security assessment
nmap --script ftp-anon,ftp-syst,ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce -sV -p 21 <target>
```

### Network-Wide FTP Discovery
```bash
# Find FTP services with anonymous access across network
nmap --script ftp-anon --open -p 21 192.168.1.0/24

# Quick FTP vulnerability scan across subnet
nmap --script ftp-vsftpd-backdoor -p 21 --open 10.0.0.0/24
```

### Custom FTP Enumeration Combinations
```bash
# Security-focused FTP scan
nmap --script "ftp-* and not ftp-brute" -sV --script-args ftp-anon.maxlist=20 -p 21 <target>

# Quick anonymous access check
nmap --script ftp-anon --script-args ftp-anon.maxlist=5 -p 21 <target>

# Vulnerability-only assessment  
nmap --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce -p 21 <target>
```

### FTP Port Variations
```bash
# Standard and alternative FTP ports
nmap --script ftp-anon -p 21,2121,8021 <target>

# High port FTP services
nmap --script ftp-* -p 21000-21100 <target>

# Multiple FTP service discovery
nmap -sV -p 21,990,2121,8021 <target>
```

---

## Complete Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: Port discovery
nmap -p 20,21,990,2121,8021 <target>

# Step 2: Service version detection
nmap -sV -p 21,990 <target>

# Step 3: Basic FTP information
nmap --script ftp-syst,banner -p 21 <target>
```

### Phase 2: Access Testing
```bash
# Step 4: Anonymous access check
nmap --script ftp-anon -p 21 <target>

# Step 5: Authentication testing (if appropriate)
nmap --script ftp-brute --script-args brute.firstonly=true -p 21 <target>

# Step 6: Directory enumeration (if access gained)
nmap --script ftp-anon --script-args ftp-anon.maxlist=1000 -p 21 <target>
```

### Phase 3: Security Assessment
```bash
# Step 7: Vulnerability scanning
nmap --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce -p 21 <target>

# Step 8: SSL/TLS assessment (if FTPS)
nmap --script ssl-enum-ciphers,ssl-cert -p 990 <target>
```

---

## Real Lab Examples

### Example 1: Anonymous FTP Server Assessment
```bash
# Target: FTP server with anonymous access

# Basic service detection
nmap -sV -p 21 192.168.1.100

# Anonymous access check with file enumeration
nmap --script ftp-anon --script-args ftp-anon.maxlist=500 -p 21 192.168.1.100

# System information gathering
nmap --script ftp-syst -p 21 192.168.1.100
```

**Expected Output:**
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3

| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 May 17  2020 pub
| -rw-r--r--    1 ftp      ftp           156 May 17  2020 welcome.txt
|_drwxrwxrwx    2 ftp      ftp          4096 May 17  2020 uploads [NSE: writeable]

| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.1.50
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

### Example 2: vsFTPd 2.3.4 Backdoor Detection
```bash
# Target: Potentially vulnerable vsftpd server

# Version detection
nmap -sV -p 21 <target>

# Backdoor detection
nmap --script ftp-vsftpd-backdoor -p 21 <target>

# If backdoor detected, test shell access
# (This would typically be followed up with netcat to port 6200)
```

### Example 3: Network FTP Discovery
```bash
# Target: Network range for FTP services

# Discovery scan for FTP services
nmap -p 21 --open 192.168.1.0/24

# Anonymous access testing on discovered services
nmap --script ftp-anon -p 21 --open 192.168.1.0/24

# Quick vulnerability assessment
nmap --script ftp-vsftpd-backdoor -p 21 --open 192.168.1.0/24
```

---

## Output Analysis and Interpretation

### Service Version Analysis

**vsFTPd 3.0.3 (Secure):**
```bash
21/tcp open  ftp     vsftpd 3.0.3
```
- **Status**: Generally secure version
- **Action**: Check configuration for anonymous access
- **Risk**: Low unless misconfigured

**vsFTPd 2.3.4 (Vulnerable):**
```bash
21/tcp open  ftp     vsftpd 2.3.4
```
- **Status**: Contains backdoor (CVE-2011-2523)  
- **Action**: Immediate update required
- **Risk**: Critical - Remote code execution

**ProFTPd 1.3.x:**
```bash
21/tcp open  ftp     ProFTPD 1.3.5a
```
- **Status**: Check for specific CVEs
- **Action**: Research version vulnerabilities
- **Risk**: Variable depending on version

### Anonymous Access Risk Assessment

**Read-Only Anonymous Access:**
```bash
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Sep 16 12:00 documents
```
- **Risk Level**: Medium
- **Impact**: Information disclosure
- **Action**: Review accessible files for sensitive data

**Write Access Anonymous:**
```bash
| drwxrwxrwx    2 0        0            4096 Sep 16 12:00 uploads [NSE: writeable]
```
- **Risk Level**: Critical
- **Impact**: File upload, potential web shell placement
- **Action**: Test upload capabilities carefully

### Vulnerability Priority Matrix

| Vulnerability | CVSS Score | Exploitability | Priority |
|---------------|------------|----------------|----------|
| vsFTPd 2.3.4 Backdoor | 10.0 | Remote/Easy | Critical |
| FTP Bounce Attack | 5.0 | Network/Medium | High |
| Anonymous Write Access | 7.5 | Remote/Easy | Critical |
| ProFTPd Backdoors | Variable | Remote/Medium | High |

---

## Integration with Other Tools

### Manual FTP Testing After Nmap
```bash
# Nmap discovery first
nmap --script ftp-anon -p 21 demo.ine.local

# Manual validation with FTP client
ftp demo.ine.local
# Name: anonymous
# Password: (leave blank or use email)
```

### Metasploit Integration
```bash
# After Nmap enumeration
nmap --script ftp-anon --open -p 21 192.168.1.0/24

# Follow up with Metasploit
msfconsole -q
use auxiliary/scanner/ftp/anonymous
set RHOSTS 192.168.1.0/24
run
```

### Automated Workflow Script
```bash
#!/bin/bash
# Comprehensive FTP enumeration pipeline

TARGET=$1
OUTPUT_DIR="ftp_enum_$(date +%Y%m%d_%H%M%S)"

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

mkdir -p $OUTPUT_DIR
echo "=== FTP Enumeration of $TARGET ==="

# Step 1: Service detection
echo "[+] Step 1: Service detection..."
nmap -sV -p 21 $TARGET -oA $OUTPUT_DIR/service_detection

# Step 2: Anonymous access testing
echo "[+] Step 2: Anonymous access testing..."
nmap --script ftp-anon --script-args ftp-anon.maxlist=20 -p 21 $TARGET -oA $OUTPUT_DIR/anonymous_test

# Step 3: System information
echo "[+] Step 3: System information gathering..."
nmap --script ftp-syst -p 21 $TARGET -oA $OUTPUT_DIR/system_info

# Step 4: Vulnerability assessment
echo "[+] Step 4: Vulnerability detection..."
nmap --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce -p 21 $TARGET -oA $OUTPUT_DIR/vulnerability_scan

# Step 5: Generate summary report
echo "[+] Step 5: Generating summary..."
echo "=== FTP Enumeration Summary ===" > $OUTPUT_DIR/summary.txt
echo "Target: $TARGET" >> $OUTPUT_DIR/summary.txt
echo "Date: $(date)" >> $OUTPUT_DIR/summary.txt
echo "" >> $OUTPUT_DIR/summary.txt

# Extract key findings
if grep -q "21/tcp.*open.*ftp" $OUTPUT_DIR/service_detection.nmap; then
    echo "✓ FTP service detected" >> $OUTPUT_DIR/summary.txt
    grep "21/tcp.*ftp" $OUTPUT_DIR/service_detection.nmap >> $OUTPUT_DIR/summary.txt
fi

if grep -q "Anonymous FTP login allowed" $OUTPUT_DIR/anonymous_test.nmap; then
    echo "✓ Anonymous access enabled" >> $OUTPUT_DIR/summary.txt
fi

if grep -q "VULNERABLE" $OUTPUT_DIR/vulnerability_scan.nmap; then
    echo "⚠ Vulnerabilities detected" >> $OUTPUT_DIR/summary.txt
fi

if grep -q "writeable" $OUTPUT_DIR/anonymous_test.nmap; then
    echo "🔴 Writable directories found" >> $OUTPUT_DIR/summary.txt
fi

echo ""
echo "Enumeration complete. Results saved in $OUTPUT_DIR/"
cat $OUTPUT_DIR/summary.txt
```

---

## Troubleshooting

### Common Issues and Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Scripts Not Found** | `NSE: failed to initialize` | Update Nmap: `nmap --script-updatedb` |
| **Connection Timeout** | Scripts hang indefinitely | Add timeout: `--script-timeout 30s` |
| **Permission Denied** | Script execution failures | Run as root: `sudo nmap` |
| **Firewall Blocking** | No response from target | Check firewall rules and connectivity |
| **False Positives** | Incorrect vulnerability reports | Manual verification required |

### Debug and Verbose Options
```bash
# Verbose script execution
nmap --script ftp-anon -v -p 21 <target>

# Script tracing for debugging
nmap --script ftp-anon --script-trace -p 21 <target>

# Debug-level output
nmap --script ftp-anon -d -p 21 <target>
```

### Performance Optimization
```bash
# Faster scanning with rate limiting
nmap --script ftp-anon --min-rate 1000 -p 21 <target>

# Adjust script timeout
nmap --script ftp-anon --script-timeout 30s -p 21 <target>

# Parallel host scanning
nmap --script ftp-anon --min-parallelism 10 -p 21 <target-range>
```

---

## Security Considerations

### Legal and Ethical Guidelines
- **Authorization Required**: Only scan systems you own or have explicit permission to test
- **Scope Compliance**: Stay within authorized IP ranges and service ports
- **Brute Force Caution**: Use ftp-brute script only with proper authorization and rate limiting
- **Data Handling**: Be careful when accessing anonymous FTP content - respect data privacy

### Operational Security
- **Logging Impact**: All FTP enumeration activities are logged by target systems
- **IDS Detection**: FTP script execution may trigger intrusion detection alerts
- **Service Impact**: Aggressive scanning may affect FTP service performance
- **Network Footprint**: Use appropriate timing templates to minimize detection

### Risk Assessment Framework

| Activity | Detection Risk | Service Impact | Legal Risk | Recommendation |
|----------|----------------|----------------|------------|----------------|
| Service Detection | Low | Minimal | Low | Generally safe with authorization |
| Anonymous Testing | Medium | Minimal | Medium | Monitor for excessive enumeration |
| Vulnerability Scanning | Medium | Minimal | Medium | Important for security assessment |
| Brute Force Testing | High | Moderate | High | Use only with explicit authorization |

### Defensive Countermeasures
- **Service Hardening**: Disable anonymous access if not required
- **Network Monitoring**: Deploy FTP-specific monitoring and alerting
- **Access Controls**: Implement proper authentication and file permissions  
- **Regular Updates**: Keep FTP server software updated with latest patches
- **Firewall Rules**: Restrict FTP access to necessary networks only

---

## eJPT Focus Points

### Essential FTP Commands for eJPT
```bash
# Basic FTP discovery and anonymous access
nmap --script ftp-anon -sV -p 21 <target>

# FTP vulnerability assessment
nmap --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor -p 21 <target>

# FTP system information
nmap --script ftp-syst -p 21 <target>

# Complete FTP enumeration (safe)
nmap --script "ftp-* and not ftp-brute" -p 21 <target>
```

### Critical Attack Vectors for eJPT

1. **Anonymous Access Exploitation**
   - File disclosure through anonymous FTP
   - Directory traversal and enumeration
   - Upload capabilities testing

2. **vsFTPd 2.3.4 Backdoor (CVE-2011-2523)**
   - Remote code execution via backdoor
   - Shell access on port 6200
   - Critical vulnerability exploitation

3. **Information Disclosure**
   - Configuration files via anonymous access
   - System information through SYST command
   - Directory structure enumeration

4. **File Upload Scenarios**
   - Web shell upload via writable FTP directories
   - Privilege escalation through file placement
   - Path traversal exploitation

### Common eJPT Scenarios

**Scenario 1: Anonymous FTP to Web Shell**
```bash
# Discovery
nmap --script ftp-anon -p 21 <target>

# Find writable web directory
# Upload web shell via FTP
# Access shell through web browser
```

**Scenario 2: vsFTPd Backdoor Exploitation**
```bash
# Detect backdoor
nmap --script ftp-vsftpd-backdoor -p 21 <target>

# Exploit backdoor (manual or Metasploit)
# Gain root shell access
```

**Scenario 3: Configuration File Discovery**
```bash
# Anonymous access
nmap --script ftp-anon --script-args ftp-anon.maxlist=100 -p 21 <target>

# Download configuration files
# Extract credentials for lateral movement
```

### Success Metrics for eJPT
- Anonymous access status determined
- File upload capabilities identified  
- Vulnerability presence confirmed
- System information extracted
- Credentials or sensitive files discovered

This comprehensive FTP enumeration guide provides all essential techniques for FTP security assessment with Nmap, focusing on practical scenarios commonly encountered in penetration testing, security audits, and eJPT certification preparation.
