# 📦 Nmap - FTP Enumeration Scripts

This section covers Nmap NSE scripts for comprehensive FTP service enumeration and security testing.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Overview](#-script-overview)
- [Prerequisites](#-prerequisites)
- [Service Detection](#-1-service-detection-and-version-scanning)
- [Anonymous Access Testing](#-2-anonymous-access-testing)
- [System Information](#-3-system-information-gathering)
- [Brute Force Testing](#-4-brute-force-testing)
- [Vulnerability Detection](#-5-vulnerability-detection)
- [Advanced Techniques](#-6-advanced-techniques)
- [Output Analysis](#-7-output-analysis-and-interpretation)
- [Integration with Other Tools](#-8-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

Nmap FTP scripts help you:
- Detect FTP service versions and configurations
- Test for anonymous FTP access and permissions
- Gather system information from FTP servers
- Perform credential brute-force attacks
- Identify FTP-specific vulnerabilities
- Enumerate FTP server capabilities and features

---

## 🗂️ Script Overview

| Script | Purpose | Auth Required | Risk Level |
|--------|---------|---------------|------------|
| `ftp-anon` | Test anonymous access | ❌ | 🟡 Medium |
| `ftp-syst` | System information | ❌ | 🟢 Low |
| `ftp-brute` | Credential brute-force | ❌ | 🔴 High |
| `ftp-vsftpd-backdoor` | vsFTPd backdoor detection | ❌ | 🟡 Medium |
| `ftp-bounce` | FTP bounce attack testing | ❌ | 🟠 High |
| `ftp-proftpd-backdoor` | ProFTPd backdoor detection | ❌ | 🟡 Medium |

---

## 📋 Prerequisites

### Nmap Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nmap

# CentOS/RHEL
sudo yum install nmap

# Verify installation and scripts
nmap --script-help ftp-*
```

### Network Requirements:
- FTP service running on target (port 21)
- Network connectivity to target
- Appropriate firewall rules

### Quick Service Check:
```bash
# Basic port scan
nmap -p 21 <target-ip>

# Quick service detection
nmap -sV -p 21 <target-ip>
```

---

## 🔧 Nmap FTP Enumeration

## 🖥️ 1. Service Detection and Version Scanning

### Basic Service Detection:
```bash
nmap -sV -p 21 <target-ip>
```

**📌 INE Lab Example:**
```bash
nmap -sV -p 21 demo.ine.local
```

**📸 Sample Output:**
```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
```

### Enhanced Service Detection:
```bash
nmap -sV -sC -p 21 <target-ip>
```

**📸 Enhanced Output:**
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

### Aggressive Service Detection:
```bash
nmap -sV -A -p 21 <target-ip>
```

---

## 🔓 2. Anonymous Access Testing

### Basic Anonymous Testing:
```bash
nmap -p 21 --script ftp-anon <target-ip>
```

**📌 INE Lab Example:**
```bash
nmap -p 21 --script ftp-anon demo.ine.local
```

**📸 Sample Output:**
```bash
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_List of FTP files and folders
```

### Detailed Anonymous Enumeration:
```bash
nmap -p 21 --script ftp-anon --script-args ftp-anon.maxlist=10 <target-ip>
```

**📸 Detailed Output:**
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

### Script Arguments for ftp-anon:
| Argument | Description | Example |
|----------|-------------|---------|
| `maxlist` | Maximum files to list | `--script-args ftp-anon.maxlist=20` |
| `maxdepth` | Directory traversal depth | `--script-args ftp-anon.maxdepth=2` |

---

## 🖥️ 3. System Information Gathering

### FTP System Information:
```bash
nmap -p 21 --script ftp-syst <target-ip>
```

**📌 INE Lab Example:**
```bash
nmap -sV -p 21 --script ftp-syst demo.ine.local
```

**📸 Sample Output:**
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

### Comprehensive System Information:
```bash
nmap -p 21 --script ftp-syst,ftp-anon -sV <target-ip>
```

---

## 🔐 4. Brute Force Testing

### Basic Brute Force:
```bash
nmap -p 21 --script ftp-brute <target-ip>
```

**⚠️ WARNING**: Brute force attacks can lock accounts and trigger security alerts.

### Advanced Brute Force with Custom Lists:
```bash
nmap -p 21 --script ftp-brute --script-args userdb=users.txt,passdb=passwords.txt <target-ip>
```

**📸 Sample Output:**
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

### Brute Force Script Arguments:
| Argument | Description | Example |
|----------|-------------|---------|
| `userdb` | Username list file | `--script-args userdb=/path/to/users.txt` |
| `passdb` | Password list file | `--script-args passdb=/path/to/passwords.txt` |
| `unpwdb.timelimit` | Time limit in seconds | `--script-args unpwdb.timelimit=30` |
| `brute.firstonly` | Stop after first success | `--script-args brute.firstonly=true` |
| `brute.threads` | Number of threads | `--script-args brute.threads=3` |

### Safe Brute Force Testing:
```bash
nmap -p 21 --script ftp-brute --script-args brute.threads=2,unpwdb.timelimit=30,brute.firstonly=true <target-ip>
```

---

## 🚨 5. Vulnerability Detection

### vsFTPd Backdoor Detection:
```bash
nmap -p 21 --script ftp-vsftpd-backdoor <target-ip>
```

**📸 Sample Output (if vulnerable):**
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

### ProFTPd Backdoor Detection:
```bash
nmap -p 21 --script ftp-proftpd-backdoor <target-ip>
```

### FTP Bounce Attack Testing:
```bash
nmap -p 21 --script ftp-bounce --script-args ftp-bounce.checkhost=<internal-ip> <target-ip>
```

**📸 Sample Output (if vulnerable):**
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

---

## 🚀 6. Advanced Techniques

### Comprehensive FTP Scanning:
```bash
nmap -p 21 --script "ftp-* and not ftp-brute" -sV <target-ip>
```

### All-in-One FTP Assessment:
```bash
nmap -p 21 --script ftp-anon,ftp-syst,ftp-vsftpd-backdoor,ftp-proftpd-backdoor -sV <target-ip>
```

### Subnet FTP Discovery:
```bash
nmap -p 21 --script ftp-anon --open <target-range>
```

**📌 Example:**
```bash
nmap -p 21 --script ftp-anon --open 192.168.1.0/24
```

### Custom Script Combinations:
```bash
# Security-focused scan
nmap -p 21 --script "ftp-* and not ftp-brute" -sV --script-args ftp-anon.maxlist=20 <target>

# Quick anonymous check
nmap -p 21 --script ftp-anon --script-args ftp-anon.maxlist=5 <target>

# Vulnerability assessment
nmap -p 21 --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor,ftp-bounce <target>
```

### Automated FTP Enumeration Script:
```bash
#!/bin/bash
# Comprehensive FTP enumeration with Nmap

TARGET=$1
OUTPUT_DIR="nmap_ftp_$(date +%Y%m%d_%H%M%S)"

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

mkdir -p $OUTPUT_DIR
echo "=== Nmap FTP Enumeration of $TARGET ==="
echo "Output directory: $OUTPUT_DIR"

# Step 1: Service detection
echo "Step 1: Service detection and version scanning..."
nmap -sV -p 21 $TARGET -oA $OUTPUT_DIR/service_detection

# Step 2: Anonymous access testing
echo "Step 2: Testing anonymous access..."
nmap -p 21 --script ftp-anon --script-args ftp-anon.maxlist=20 $TARGET -oA $OUTPUT_DIR/anonymous_test

# Step 3: System information
echo "Step 3: Gathering system information..."
nmap -p 21 --script ftp-syst $TARGET -oA $OUTPUT_DIR/system_info

# Step 4: Vulnerability scanning
echo "Step 4: Vulnerability detection..."
nmap -p 21 --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor $TARGET -oA $OUTPUT_DIR/vulnerability_scan

# Step 5: Comprehensive scan
echo "Step 5: Comprehensive FTP script scan..."
nmap -p 21 --script "ftp-* and not ftp-brute" -sV $TARGET -oA $OUTPUT_DIR/comprehensive

# Step 6: Generate summary
echo "Step 6: Generating summary..."
echo "=== FTP Enumeration Summary ===" > $OUTPUT_DIR/summary.txt
echo "Target: $TARGET" >> $OUTPUT_DIR/summary.txt
echo "Date: $(date)" >> $OUTPUT_DIR/summary.txt
echo >> $OUTPUT_DIR/summary.txt

# Extract key findings
if grep -q "ftp.*open" $OUTPUT_DIR/service_detection.nmap; then
    echo "✓ FTP service detected" >> $OUTPUT_DIR/summary.txt
    grep "ftp.*vsftpd\|ProFTPd\|FileZilla" $OUTPUT_DIR/service_detection.nmap >> $OUTPUT_DIR/summary.txt
fi

if grep -q "Anonymous FTP login allowed" $OUTPUT_DIR/anonymous_test.nmap; then
    echo "✓ Anonymous access enabled" >> $OUTPUT_DIR/summary.txt
fi

if grep -q "VULNERABLE" $OUTPUT_DIR/vulnerability_scan.nmap; then
    echo "⚠ Vulnerabilities detected" >> $OUTPUT_DIR/summary.txt
fi

echo "Enumeration complete. Results saved in $OUTPUT_DIR/"
cat $OUTPUT_DIR/summary.txt
```

---

## 🔍 7. Output Analysis and Interpretation

### Service Version Analysis:

**vsFTPd Detection:**
```bash
21/tcp open  ftp     vsftpd 3.0.3
```
- **Software**: Very Secure FTP Daemon (Linux)
- **Version**: 3.0.3 - check for CVEs
- **Security**: Generally secure implementation

**ProFTPd Detection:**
```bash
21/tcp open  ftp     ProFTPD 1.3.5a
```
- **Software**: Professional FTP daemon
- **Version**: 1.3.5a - research vulnerabilities
- **Security**: History of security issues

**FileZilla Server:**
```bash
21/tcp open  ftp     FileZilla ftpd 0.9.60
```
- **Software**: FileZilla Server (Windows)
- **Version**: 0.9.60 - check for updates
- **Platform**: Windows-based FTP server

### Anonymous Access Analysis:

**Full Anonymous Access:**
```bash
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 0        0            4096 Sep 16 12:00 uploads [NSE: writeable]
```
- **Risk**: High - write access allows file upload
- **Action**: Test upload capabilities carefully

**Read-Only Anonymous Access:**
```bash
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Sep 16 12:00 documents
```
- **Risk**: Medium - information disclosure
- **Action**: Enumerate and download files

### Vulnerability Analysis:

**vsFTPd Backdoor (CVE-2011-2523):**
```bash
|   VULNERABLE:
|   vsFTPd version 2.3.4 backdoor
|     State: VULNERABLE (Exploitable)
```
- **Impact**: Remote code execution
- **CVSS**: 10.0 (Critical)
- **Action**: Immediate patching required

### Directory Permission Interpretation:

| Permission | Owner | Group | Other | Meaning |
|------------|-------|-------|--------|---------|
| `drwxrwxrwx` | rwx | rwx | rwx | Fully writable directory |
| `drwxr-xr-x` | rwx | r-x | r-x | Read-only for group/other |
| `-rw-r--r--` | rw- | r-- | r-- | Read-only file |
| `-rw-rw-rw-` | rw- | rw- | rw- | Writable file |

---

## 🔗 8. Integration with Other Tools

### With Metasploit:
```bash
# Nmap discovery
nmap -p 21 --script ftp-anon --open 192.168.1.0/24

# Follow up with Metasploit
use auxiliary/scanner/ftp/anonymous
set RHOSTS <discovered-hosts>
run
```

### With Manual Tools:
```bash
# Nmap enumeration
nmap -p 21 --script ftp-anon,ftp-syst demo.ine.local

# Manual validation
ftp demo.ine.local
# Name: anonymous
# Password: (blank)
```

### Pipeline Integration:
```bash
#!/bin/bash
# Nmap FTP to manual enumeration pipeline

TARGET=$1

echo "=== FTP Enumeration Pipeline ==="

# Step 1: Nmap service detection
echo "Step 1: Service detection"
if nmap -p 21 $TARGET | grep -q "21/tcp open"; then
    echo "✓ FTP service detected"
    
    # Step 2: Version and script scanning
    echo "Step 2: Detailed enumeration"
    nmap -sV -p 21 --script "ftp-* and not ftp-brute" $TARGET
    
    # Step 3: Check for anonymous access
    if nmap -p 21 --script ftp-anon $TARGET | grep -q "Anonymous FTP login allowed"; then
        echo "✓ Anonymous access detected"
        echo "Step 3: Testing manual access"
        
        # Automated anonymous test
        echo -e "anonymous\nanonymous\nls\nquit" | ftp $TARGET
    fi
    
    # Step 4: Check for vulnerabilities
    echo "Step 4: Vulnerability check"
    nmap -p 21 --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor $TARGET
    
else
    echo "✗ No FTP service detected"
fi
```

### Cross-Validation:
```bash
# Compare Nmap and Metasploit results
nmap -p 21 --script ftp-anon <target> > nmap_results.txt
msfconsole -q -x "use auxiliary/scanner/ftp/anonymous; set RHOSTS <target>; run; exit" > msf_results.txt

# Validate with manual tools
curl ftp://<target>/ > curl_results.txt
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Scripts Not Found** | `NSE: failed to initialize the script engine` | Update Nmap or check script location |
| **Connection Timeout** | Scripts hang or timeout | Increase timeout with `--script-timeout` |
| **Permission Denied** | `SCRIPT ENGINE: Aborting script scan` | Run as root for some operations |
| **Firewall Blocking** | No response from scripts | Check firewall rules and NAT |
| **Script Errors** | Script execution failures | Update NSE script database |

### Debugging Commands:
```bash
# Check script location
locate ftp-anon.nse

# Update script database
nmap --script-updatedb

# Verbose script execution
nmap -p 21 --script ftp-anon --script-trace <target>

# Debug script execution
nmap -p 21 --script ftp-anon -d <target>
```

### Performance Optimization:
```bash
# Faster scanning
nmap -p 21 --script ftp-anon --min-rate 1000 <target>

# Adjust timeouts
nmap -p 21 --script ftp-anon --script-timeout 30s <target>

# Parallel host scanning
nmap -p 21 --script ftp-anon --min-parallelism 10 <target-range>
```

### Script Customization:
```bash
# Custom script arguments
nmap -p 21 --script ftp-anon --script-args ftp-anon.maxlist=50,ftp-anon.maxdepth=3 <target>

# Combine with timing templates
nmap -T4 -p 21 --script "ftp-* and not ftp-brute" <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only scan systems you own or have explicit permission to test
- **Scope Compliance**: Stay within authorized IP ranges and ports
- **Brute Force Caution**: Use ftp-brute script only with proper authorization
- **Data Sensitivity**: Be careful when accessing anonymous FTP content

### Operational Security:
- **Logging Impact**: Nmap scans are logged by target systems
- **Network Detection**: FTP script execution may trigger IDS alerts
- **Service Disruption**: Aggressive scanning may affect FTP service performance
- **Timing**: Use appropriate timing templates to avoid detection

### Detection Indicators:
- **Network Logs**: Port scanning patterns in firewall logs
- **FTP Server Logs**: Connection attempts from scanning IP
- **IDS Signatures**: Nmap script execution patterns
- **Security Alerts**: Unusual FTP enumeration activity

### Defensive Countermeasures:
- **Disable Unnecessary Services**: Remove FTP if not required
- **Anonymous Access**: Disable anonymous login if not needed
- **Network Monitoring**: Deploy FTP-specific monitoring rules
- **Regular Updates**: Keep FTP server software updated
- **Access Controls**: Implement proper authentication and authorization

### Risk Assessment:
| Activity | Detection Risk | Impact | Recommendation |
|----------|----------------|--------|----------------|
| Service Detection | 🟢 Low | 🟢 Low | Generally safe with authorization |
| Anonymous Testing | 🟡 Medium | 🟡 Medium | Monitor for excessive enumeration |
| Brute Force Scripts | 🔴 High | 🔴 High | Use only with explicit authorization |
| Vulnerability Scripts | 🟡 Medium | 🟠 High | Important for security assessment |

---

## 📖 References

### Official Documentation:
- [Nmap NSE Documentation](https://nmap.org/nsedoc/) - Complete NSE script reference
- [Nmap FTP Scripts](https://nmap.org/nsedoc/categories/ftp.html) - FTP-specific scripts
- [NSE Script Development](https://nmap.org/book/nse-tutorial.html) - Script development guide

### Security Research:
- [FTP Security Assessment](https://www.sans.org/reading-room/whitepapers/protocols/ftp-security-1327) - SANS methodology
- [FTP Vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=ftp) - CVE database
- [Network Service Enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp) - Practical techniques

### Learning Resources:
- [Nmap Network Scanning](https://nmap.org/book/) - Official Nmap guide
- [NSE Script Examples](https://github.com/nmap/nmap/tree/master/scripts) - Script source code
- [Network Discovery and Enumeration](https://www.offensive-security.com/metasploit-unleashed/) - Advanced techniques

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Service detection
nmap -sV -p 21 <target>

# Anonymous testing
nmap -p 21 --script ftp-anon <target>

# System information
nmap -p 21 --script ftp-syst <target>

# Comprehensive scan
nmap -p 21 --script "ftp-* and not ftp-brute" -sV <target>
```

### Key NSE Scripts:
- **ftp-anon**: Anonymous access testing
- **ftp-syst**: System information gathering
- **ftp-vsftpd-backdoor**: vsFTPd backdoor detection
- **ftp-brute**: Credential brute-force (use carefully)

### Script Arguments:
```bash
# ftp-anon arguments
--script-args ftp-anon.maxlist=20,ftp-anon.maxdepth=2

# ftp-brute arguments
--script-args userdb=users.txt,passdb=passwords.txt,brute.firstonly=true
```

### Output Priorities:
1. **Critical**: VULNERABLE findings from backdoor scripts
2. **High**: Anonymous write access (writable directories)
3. **Medium**: Anonymous read access, system information
4. **Low**: Version information, connection details

### Success Indicators:
- FTP service version identified
- Anonymous access status determined
- System information gathered
- Vulnerability status assessed
- Directory permissions analyzed

### Time Estimates:
- **Service detection**: 10-30 seconds
- **Anonymous testing**: 30 seconds - 2 minutes
- **System information**: 30 seconds - 1 minute
- **Comprehensive scan**: 2-5 minutes
- **Large network scan**: 10+ minutes

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting FTP enumeration with Nmap.*
