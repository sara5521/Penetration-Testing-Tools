# 🎯 Metasploit - SMB Version Detection

This section covers the Metasploit auxiliary module for SMB version detection and fingerprinting.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Module Overview](#-module-overview)
- [Prerequisites](#-prerequisites)
- [Basic Usage](#-1-basic-smb-version-detection)
- [Advanced Configuration](#-2-advanced-configuration)
- [Output Analysis](#-3-output-analysis-and-interpretation)
- [Multi-Target Scanning](#-4-multi-target-scanning)
- [Integration with Other Tools](#-5-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

The `auxiliary/scanner/smb/smb_version` module helps you:
- Identify the exact version of SMB/Samba services
- Detect supported SMB dialects (SMBv1, SMBv2, SMBv3)
- Gather OS and hostname information
- Profile target systems for vulnerability assessment
- Plan follow-up enumeration and exploitation

---

## 🗂️ Module Overview

| Attribute | Details |
|-----------|---------|
| **Module Path** | `auxiliary/scanner/smb/smb_version` |
| **Type** | Scanner |
| **Disclosure Date** | N/A |
| **Author** | hdm, Spencer McIntyre |
| **Platform** | All |
| **Targets** | SMB/CIFS services |
| **Default Port** | 445/tcp |

---

## 📋 Prerequisites

### Environment Setup:
```bash
# Start Metasploit
msfconsole -q

# Verify module availability
search smb_version
```

### Target Requirements:
- SMB service running on target (port 445 or 139)
- Network connectivity to target
- No authentication required for version detection

### Quick Connectivity Test:
```bash
# Test SMB port accessibility
nmap -p 445 <target-ip>
```

---

## 🔧 Metasploit SMB Version Detection

## 🖥️ 1. Basic SMB Version Detection

### Module Loading:
```bash
use auxiliary/scanner/smb/smb_version
show options
```

### Basic Configuration:
```bash
set RHOSTS <target-ip>
run
```

**📌 INE Lab Example:**
```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS demo.ine.local
run
```

**📸 Sample Output:**
```bash
[*] 192.220.69.3:445 - SMB Detected (versions: 1, 2, 3)
[*] 192.220.69.3:445 - Host is running: Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] 192.220.69.3:445 - Negotiated dialect: NT LM 0.12 (SMBv1)
[*] 192.220.69.3:445 - Native OS: Unix
[*] 192.220.69.3:445 - Native LanMan: Samba
[*] Auxiliary module execution completed
```

**Interpretation:**
- 🖥️ **SMB Versions**: `1, 2, 3` → Multiple protocol versions supported
- 💻 **Host OS**: `Windows 6.1 (Samba 4.3.11-Ubuntu)` → Linux running Samba
- 🔄 **Negotiated Dialect**: `NT LM 0.12` → SMBv1 protocol used
- 🐧 **Native OS**: `Unix` → Confirms Linux/Unix system
- 📦 **LanMan**: `Samba` → Open-source SMB implementation

---

## ⚙️ 2. Advanced Configuration

### Common Options:
```bash
show options
```

| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `RHOSTS` | Target host(s) | - | `192.168.1.100` |
| `RPORT` | Target port | `445` | `139` |
| `THREADS` | Scan threads | `1` | `10` |
| `ConnectTimeout` | Connection timeout | `10` | `5` |
| `SendTimeout` | Send timeout | `5` | `3` |

### Advanced Usage Examples:

**🌐 Different Port:**
```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS <target-ip>
set RPORT 139
run
```

**⚡ Faster Scanning:**
```bash
set RHOSTS <target-ip>
set ConnectTimeout 5
set SendTimeout 3
run
```

**🔍 Verbose Output:**
```bash
set VERBOSE true
set RHOSTS <target-ip>
run
```

---

## 🔍 3. Output Analysis and Interpretation

### Understanding SMB Version Information:

**🔢 SMB Protocol Versions:**
- **SMBv1 (1)** → Legacy protocol, security risks
- **SMBv2 (2)** → Improved security and performance
- **SMBv3 (3)** → Latest with encryption support

**🖥️ OS Fingerprinting:**
- **Windows 6.1** → Windows 7/Server 2008 R2
- **Samba x.x.x** → Linux/Unix SMB implementation
- **Native OS: Unix/Windows** → Actual operating system

### Security Risk Assessment:

| Finding | Risk Level | Implications |
|---------|------------|-------------|
| SMBv1 Enabled | 🔴 Critical | Vulnerable to EternalBlue (MS17-010) |
| Samba < 4.0 | 🟠 High | Multiple known vulnerabilities |
| Windows 6.1 | 🟡 Medium | End-of-support OS version |
| Mixed Versions | 🟡 Medium | Potential compatibility issues |

**🚨 Critical Vulnerabilities to Check:**
- **MS17-010 (EternalBlue)** - SMBv1 RCE
- **CVE-2017-7494 (SambaCry)** - Samba RCE
- **MS08-067** - SMB vulnerability

### Sample Analysis Scenarios:

**Scenario 1: Windows Server**
```bash
[*] 10.0.0.100:445 - SMB Detected (versions: 2, 3)
[*] 10.0.0.100:445 - Host is running: Windows Server 2019 Standard
```
**Analysis:** Modern Windows server, SMBv1 disabled, lower risk

**Scenario 2: Legacy System**
```bash
[*] 10.0.0.50:445 - SMB Detected (versions: 1)
[*] 10.0.0.50:445 - Host is running: Windows XP
```
**Analysis:** High-risk legacy system, SMBv1 only, likely vulnerable

**Scenario 3: Samba Server**
```bash
[*] 10.0.0.200:445 - SMB Detected (versions: 1, 2, 3)
[*] 10.0.0.200:445 - Host is running: Unix (Samba 4.3.11-Ubuntu)
```
**Analysis:** Linux Samba server, check for Samba-specific CVEs

---

## 🌐 4. Multi-Target Scanning

### IP Range Scanning:
```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.1-50
set THREADS 10
run
```

### Subnet Scanning:
```bash
set RHOSTS 192.168.1.0/24
set THREADS 20
run
```

### File-Based Target List:
```bash
# Create target list file
echo -e "10.0.0.1\n10.0.0.5\n10.0.0.10" > targets.txt

# Use file in Metasploit
set RHOSTS file:targets.txt
set THREADS 5
run
```

### Database Integration:
```bash
# Enable workspace
workspace -a smb_assessment

# Run scan (results saved automatically)
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.0/24
set THREADS 15
run

# Query results
hosts
services -p 445
```

**📊 Sample Multi-Target Output:**
```bash
[*] 192.168.1.10:445 - SMB Detected (versions: 1, 2, 3)
[*] 192.168.1.10:445 - Host is running: Windows 7 Professional
[*] 192.168.1.15:445 - SMB Detected (versions: 2, 3)
[*] 192.168.1.15:445 - Host is running: Windows 10 Pro
[*] 192.168.1.20:445 - SMB Detected (versions: 1, 2, 3)
[*] 192.168.1.20:445 - Host is running: Unix (Samba 4.6.2)
```

**Analysis Summary:**
- 192.168.1.10 → Windows 7, SMBv1 enabled (high risk)
- 192.168.1.15 → Windows 10, SMBv1 disabled (lower risk)  
- 192.168.1.20 → Linux Samba, mixed versions (check CVEs)

---

## 🔗 5. Integration with Other Tools

### Combining with Nmap:
```bash
# Nmap SMB discovery
nmap -p 445 --script smb-os-discovery 192.168.1.0/24

# Then use Metasploit for detailed version info
use auxiliary/scanner/smb/smb_version
set RHOSTS <discovered-hosts>
run
```

### Follow-up Metasploit Modules:
```bash
# After version detection, enumerate shares
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS <target>
run

# Test for vulnerabilities based on version
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS <target>
run

# Attempt authentication
use auxiliary/scanner/smb/smb_login
set RHOSTS <target>
set USER_FILE usernames.txt
set PASS_FILE passwords.txt
run
```

### External Tool Integration:
```bash
# enum4linux for comprehensive enumeration
enum4linux -a <target>

# smbclient for share access
smbclient -L <target> -N

# crackmapexec for authentication testing
crackmapexec smb <target> -u '' -p ''
```

### Automated Workflow:
```bash
# Create automated SMB assessment script
echo '#!/bin/bash
echo "=== SMB Version Detection ==="
msfconsole -q -x "
use auxiliary/scanner/smb/smb_version;
set RHOSTS $1;
set THREADS 5;
run;
exit"

echo "=== Nmap SMB Scripts ==="
nmap -p 445 --script "smb-os-discovery,smb-security-mode" $1' > smb_assess.sh

chmod +x smb_assess.sh
./smb_assess.sh 192.168.1.100
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Refused** | `Connection refused (10061)` | Check if SMB service is running |
| **Timeout** | Module hangs or times out | Reduce timeout values or check network |
| **Permission Denied** | `Access denied` | Normal for version detection, continue |
| **No Response** | No output from target | SMB may be disabled or firewalled |
| **Module Not Found** | `Failed to load module` | Update Metasploit framework |

### Debugging Commands:
```bash
# Test basic connectivity
use auxiliary/scanner/portscan/tcp
set RHOSTS <target>
set PORTS 139,445
run

# Enable verbose debugging
set VERBOSE true
set RHOSTS <target>
run

# Check Metasploit version
version

# Update framework if needed
msfupdate
```

### Common Error Messages:
```bash
# Connection issues
[*] 192.168.1.100:445 - The connection was refused by the remote host (10.0.0.100:445).
# Solution: SMB service not running or blocked

# Timeout errors  
[*] 192.168.1.100:445 - The connection timed out (10.0.0.100:445).
# Solution: Reduce timeout or check network connectivity

# Protocol errors
[*] 192.168.1.100:445 - The server responded with error: STATUS_NOT_SUPPORTED
# Solution: Normal response, version may still be detected
```

### Performance Optimization:
```bash
# Faster scanning for large networks
set THREADS 20
set ConnectTimeout 3
set SendTimeout 2
run

# Memory optimization
unset VERBOSE
set WORKSPACE false
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only scan systems you own or have permission to test
- **Scope Compliance**: Stay within authorized IP ranges and ports
- **Documentation**: Maintain detailed logs of all scanning activities
- **Responsible Disclosure**: Report vulnerabilities through proper channels

### Operational Security:
- **Logging Awareness**: Version detection creates minimal logs but is still recorded
- **Network Detection**: Multiple rapid connections may trigger IDS alerts
- **Timing**: Consider slower scans to avoid detection
- **Proxy Usage**: Use SOCKS proxies for additional anonymization

### Detection Indicators:
SMB version detection may be detected through:
- **Connection Logs**: Multiple SMB connection attempts
- **Protocol Analysis**: SMB negotiation packets
- **Timing Patterns**: Rapid sequential connections
- **Source IP**: Connections from unusual/unauthorized IPs

### Risk Assessment:
| Activity | Detection Risk | Impact | Recommendation |
|----------|----------------|--------|----------------|
| Single Host Scan | 🟢 Low | 🟢 Low | Generally safe |
| Subnet Scan | 🟡 Medium | 🟡 Medium | Use slower timing |
| Large Network Scan | 🔴 High | 🟠 High | Coordinate with IT team |

---

## 📖 References

### Official Documentation:
- [Metasploit Framework](https://docs.rapid7.com/metasploit/) - Official documentation
- [SMB Protocol Specifications](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/) - Microsoft SMB docs
- [Samba Documentation](https://www.samba.org/samba/docs/) - Open source SMB implementation

### Security Research:
- [MS17-010 EternalBlue](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144) - Critical SMBv1 vulnerability
- [SambaCry CVE-2017-7494](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7494) - Samba RCE
- [SMB Security Best Practices](https://www.sans.org/reading-room/whitepapers/protocols/applying-host-based-security-layered-approach-securing-microsoft-systems-36057) - SANS guidance

### Learning Resources:
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) - Free training course
- [Penetration Testing with Metasploit](https://www.packtpub.com/product/mastering-metasploit/9781788623179) - Advanced techniques
- [SMB Enumeration Guide](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) - Comprehensive methodology

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Basic version detection
use auxiliary/scanner/smb/smb_version; set RHOSTS <target>; run

# Fast subnet scan
use auxiliary/scanner/smb/smb_version; set RHOSTS <subnet>; set THREADS 10; run

# Port 139 scan
use auxiliary/scanner/smb/smb_version; set RHOSTS <target>; set RPORT 139; run
```

### Follow-up Modules:
1. `auxiliary/scanner/smb/smb_enumshares` - Share enumeration
2. `auxiliary/scanner/smb/smb_login` - Authentication testing  
3. `auxiliary/scanner/smb/smb_ms17_010` - EternalBlue check
4. `auxiliary/scanner/smb/smb_enumusers` - User enumeration

### Key Information to Collect:
- Exact SMB/Samba version strings
- Supported SMB protocol versions
- Operating system information
- Hostname and domain details
- Security configuration indicators

### Time Estimates:
- **Single host**: 10-30 seconds
- **Small subnet (/28)**: 2-5 minutes  
- **Large subnet (/24)**: 10-30 minutes
- **Enterprise network**: 1+ hours

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting security assessments.*
