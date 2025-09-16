# 🛠️ FTP CLI Tools - Command Line Enumeration

This section covers various command-line tools for FTP service enumeration and interaction.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Tools Overview](#-tools-overview)
- [Prerequisites](#-prerequisites)
- [Banner Grabbing](#-1-banner-grabbing)
- [Anonymous Access Testing](#-2-anonymous-access-testing)
- [FTP Client Operations](#-3-ftp-client-operations)
- [Advanced CLI Techniques](#-4-advanced-cli-techniques)
- [Automated Enumeration](#-5-automated-enumeration)
- [Output Analysis](#-6-output-analysis-and-interpretation)
- [Integration with Other Tools](#-7-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

FTP CLI tools help you:
- Grab FTP server banners and version information
- Test for anonymous access to FTP services
- Enumerate directories and files on FTP servers
- Download and upload files via FTP
- Test FTP security configurations
- Perform reconnaissance on FTP services
- Validate FTP service availability and functionality

---

## 🗂️ Tools Overview

| Tool | Purpose | Authentication | Output Format |
|------|---------|----------------|---------------|
| `telnet` | Banner grabbing, raw FTP commands | Manual | Plain text |
| `nc` (netcat) | Quick banner grab, connection test | Manual | Plain text |
| `ftp` | Interactive FTP client | Interactive | Structured |
| `curl` | HTTP-style FTP operations | Command-line | Structured |
| `wget` | File downloads from FTP | Command-line | Progress output |
| `lftp` | Advanced FTP client | Interactive/Script | Rich output |

---

## 📋 Prerequisites

### Tool Installation:
```bash
# Ubuntu/Debian - most tools are pre-installed
sudo apt update
sudo apt install ftp curl wget netcat-openbsd lftp

# CentOS/RHEL
sudo yum install ftp curl wget nc lftp

# Verify installations
which ftp telnet nc curl wget lftp
```

### Network Requirements:
- FTP service running on target (port 21)
- Network connectivity to target
- Firewall rules allowing FTP traffic

### Quick Service Check:
```bash
# Test FTP port availability
nmap -p 21 <target-ip>

# Quick connection test
timeout 5 nc -zv <target-ip> 21
```

---

## 🔧 FTP CLI Enumeration

## 🏷️ 1. Banner Grabbing

### Using Telnet:
```bash
telnet <target-ip> 21
```

**📌 INE Lab Example:**
```bash
telnet demo.ine.local 21
```

**📸 Sample Output:**
```bash
Trying 192.220.69.3...
Connected to demo.ine.local.
Escape character is '^]'.
220 (vsFTPd 3.0.3)
```

**Commands in telnet session:**
```bash
# Get system information
SYST

# Get help
HELP

# Quit
QUIT
```

### Using Netcat:
```bash
nc <target-ip> 21
```

**📌 INE Lab Example:**
```bash
nc demo.ine.local 21
```

**📸 Sample Output:**
```bash
220 (vsFTPd 3.0.3)
```

### Using Curl for Banner:
```bash
curl -v ftp://<target-ip>/
```

**📸 Sample Output:**
```bash
* Connected to demo.ine.local (192.220.69.3) port 21 (#0)
< 220 (vsFTPd 3.0.3)
> USER anonymous
< 331 Please specify the password.
> PASS ftp@example.com
< 230 Login successful.
```

### Automated Banner Grabbing:
```bash
#!/bin/bash
# FTP banner grabbing script

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== FTP Banner Grabbing for $TARGET ==="
echo "Date: $(date)"
echo

# Method 1: Using nc with timeout
echo "Method 1: Netcat banner grab"
timeout 5 nc $TARGET 21 2>/dev/null | head -3

echo
echo "Method 2: Telnet banner grab"
# Method 2: Using expect with telnet (if available)
(echo "QUIT"; sleep 1) | telnet $TARGET 21 2>/dev/null | grep -E "220|Connected"

echo
echo "Method 3: Curl verbose"
# Method 3: Using curl
timeout 10 curl -v --connect-timeout 5 ftp://$TARGET/ 2>&1 | grep -E "220|Connected" | head -3
```

---

## 🔓 2. Anonymous Access Testing

### Basic Anonymous Login Test:
```bash
ftp <target-ip>
# At prompts:
# Name: anonymous
# Password: anonymous (or leave blank)
```

**📌 INE Lab Example:**
```bash
ftp demo.ine.local
Connected to demo.ine.local.
220 (vsFTPd 3.0.3)
Name (demo.ine.local:user): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

### Using Curl for Anonymous Access:
```bash
curl ftp://<target-ip>/
curl ftp://anonymous:anonymous@<target-ip>/
```

**📸 Sample Output:**
```bash
drwxr-xr-x    2 0        0            4096 Sep 16 12:00 pub
-rw-r--r--    1 0        0             234 Sep 16 12:00 welcome.txt
```

### Automated Anonymous Test:
```bash
#!/bin/bash
# Test anonymous FTP access

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Testing Anonymous FTP Access on $TARGET ==="

# Test with curl
echo "Testing with curl..."
if timeout 10 curl -s ftp://$TARGET/ >/dev/null 2>&1; then
    echo "✓ Anonymous access successful"
    echo "Directory listing:"
    curl -s ftp://$TARGET/ | head -10
else
    echo "✗ Anonymous access denied or FTP not available"
fi

echo
# Test with ftp client
echo "Testing with ftp client..."
cat > /tmp/ftp_test.txt << EOF
open $TARGET
user anonymous anonymous
ls
quit
EOF

ftp -n < /tmp/ftp_test.txt > /tmp/ftp_result.txt 2>&1

if grep -q "230 Login successful" /tmp/ftp_result.txt; then
    echo "✓ Anonymous login successful with ftp client"
    grep -A 10 "200 PORT command successful" /tmp/ftp_result.txt
else
    echo "✗ Anonymous login failed"
    grep -E "530|Login incorrect" /tmp/ftp_result.txt
fi

rm -f /tmp/ftp_test.txt /tmp/ftp_result.txt
```

---

## 💾 3. FTP Client Operations

### Interactive FTP Session Commands:

```bash
ftp <target-ip>
```

**Navigation Commands:**
```bash
ftp> ls                    # List current directory
ftp> dir                   # Detailed directory listing
ftp> cd dirname            # Change directory
ftp> pwd                   # Print working directory
ftp> cdup                  # Change to parent directory
```

**File Transfer Commands:**
```bash
ftp> get filename          # Download file
ftp> mget *.txt           # Download multiple files
ftp> put localfile        # Upload file
ftp> mput *.doc          # Upload multiple files
```

**System Information:**
```bash
ftp> syst                 # System information
ftp> stat                 # Connection status
ftp> remotehelp          # Server help
```

**Settings Commands:**
```bash
ftp> binary              # Set binary mode
ftp> ascii               # Set ASCII mode
ftp> prompt              # Toggle prompting for mget/mput
ftp> passive             # Toggle passive mode
```

**📌 Sample Interactive Session:**
```bash
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Sep 16 12:00 documents
drwxr-xr-x    2 0        0            4096 Sep 16 12:00 pub
-rw-r--r--    1 0        0             156 Sep 16 12:00 readme.txt
226 Directory send OK.

ftp> cd documents
250 Directory successfully changed.

ftp> get important.txt
local: important.txt remote: important.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for important.txt (1024 bytes).
226 Transfer complete.
1024 bytes received in 0.00 secs (1.2 MB/s)

ftp> quit
221 Goodbye.
```

### Non-Interactive FTP Operations:

**Batch Commands:**
```bash
# Create command file
cat > ftp_commands.txt << EOF
open demo.ine.local
user anonymous anonymous
binary
cd pub
ls
mget *.txt
quit
EOF

# Execute batch
ftp -n < ftp_commands.txt
```

**Single Command Execution:**
```bash
echo -e "user anonymous anonymous\nls\nquit" | ftp -n demo.ine.local
```

---

## 🚀 4. Advanced CLI Techniques

### Using LFTP (Advanced FTP Client):
```bash
# Connect with lftp
lftp ftp://anonymous:anonymous@<target-ip>

# Advanced lftp commands
lftp> ls -la              # Detailed listing
lftp> find . -name "*.txt"  # Find files
lftp> mirror              # Synchronize directories
lftp> pget -n 4 largefile # Parallel download
lftp> queue               # Show download queue
```

### Wget FTP Downloads:
```bash
# Download single file
wget ftp://anonymous:anonymous@<target-ip>/file.txt

# Download recursively
wget -r ftp://anonymous:anonymous@<target-ip>/

# Download with custom user agent
wget --user-agent="Mozilla/5.0" ftp://anonymous:anonymous@<target-ip>/file.txt

# Continue partial downloads
wget -c ftp://anonymous:anonymous@<target-ip>/largefile.zip
```

### Curl Advanced FTP Operations:
```bash
# Upload file
curl -T localfile.txt ftp://user:pass@<target-ip>/

# Download with resume
curl -C - ftp://anonymous:@<target-ip>/largefile.zip -o largefile.zip

# Custom FTP commands
curl -Q "SITE CHMOD 755 file.txt" ftp://user:pass@<target-ip>/

# List directory in different formats
curl ftp://anonymous:@<target-ip>/ --list-only
```

### Raw FTP Commands via Netcat:
```bash
#!/bin/bash
# Raw FTP command execution

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Raw FTP Commands via Netcat ==="

# Send raw FTP commands
(
echo "USER anonymous"
sleep 1
echo "PASS anonymous"
sleep 1
echo "SYST"
sleep 1
echo "PWD"
sleep 1
echo "LIST"
sleep 1
echo "QUIT"
) | nc $TARGET 21
```

---

## 🤖 5. Automated Enumeration

### Comprehensive FTP Enumeration Script:
```bash
#!/bin/bash
# Comprehensive FTP enumeration

TARGET=$1
OUTPUT_DIR="ftp_enum_$(date +%Y%m%d_%H%M%S)"

if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

mkdir -p $OUTPUT_DIR
echo "=== FTP Enumeration of $TARGET ==="
echo "Output directory: $OUTPUT_DIR"
echo

# Step 1: Banner grabbing
echo "Step 1: Banner grabbing..."
timeout 5 nc $TARGET 21 > $OUTPUT_DIR/banner.txt 2>&1
echo "Banner: $(cat $OUTPUT_DIR/banner.txt | head -1)"

# Step 2: Anonymous access test
echo "Step 2: Testing anonymous access..."
cat > $OUTPUT_DIR/ftp_anon_test.ftp << EOF
open $TARGET
user anonymous anonymous
syst
pwd
ls -la
quit
EOF

ftp -n < $OUTPUT_DIR/ftp_anon_test.ftp > $OUTPUT_DIR/anonymous_test.txt 2>&1

if grep -q "230 Login successful" $OUTPUT_DIR/anonymous_test.txt; then
    echo "✓ Anonymous access successful"
    
    # Step 3: Directory enumeration
    echo "Step 3: Directory enumeration..."
    cat > $OUTPUT_DIR/ftp_enum.ftp << EOF
open $TARGET
user anonymous anonymous
binary
pwd
ls -la
cd /
ls -la
cd pub
ls -la
cd ../etc
ls -la
cd ../var
ls -la
quit
EOF
    
    ftp -n < $OUTPUT_DIR/ftp_enum.ftp > $OUTPUT_DIR/directory_enum.txt 2>&1
    
    # Step 4: File download
    echo "Step 4: Downloading interesting files..."
    mkdir -p $OUTPUT_DIR/downloads
    
    # Extract filenames from directory listing
    grep "^-" $OUTPUT_DIR/directory_enum.txt | awk '{print $9}' | grep -E "\.(txt|log|conf|ini|xml)$" > $OUTPUT_DIR/interesting_files.txt
    
    if [ -s $OUTPUT_DIR/interesting_files.txt ]; then
        echo "Found interesting files:"
        cat $OUTPUT_DIR/interesting_files.txt
        
        # Download small files
        while read filename; do
            echo "Downloading $filename..."
            curl -s -o "$OUTPUT_DIR/downloads/$filename" "ftp://anonymous:@$TARGET/$filename"
        done < $OUTPUT_DIR/interesting_files.txt
    fi
    
else
    echo "✗ Anonymous access denied"
fi

# Step 5: Vulnerability checks
echo "Step 5: Basic vulnerability checks..."

# Check for common vulnerabilities
echo "Checking for bounce attacks..."
(echo "PORT 127,0,0,1,0,80"; sleep 1; echo "QUIT") | nc $TARGET 21 > $OUTPUT_DIR/bounce_test.txt 2>&1

echo "Enumeration complete. Results saved in $OUTPUT_DIR/"

# Summary
echo
echo "=== Summary ==="
echo "Banner: $(cat $OUTPUT_DIR/banner.txt | head -1)"
if grep -q "230 Login successful" $OUTPUT_DIR/anonymous_test.txt; then
    echo "Anonymous access: ✓ Enabled"
    file_count=$(ls -1 $OUTPUT_DIR/downloads/ 2>/dev/null | wc -l)
    echo "Files downloaded: $file_count"
else
    echo "Anonymous access: ✗ Disabled"
fi
```

### Multi-Target FTP Scanner:
```bash
#!/bin/bash
# Scan multiple targets for FTP

TARGETS_FILE=$1
if [ ! -f "$TARGETS_FILE" ]; then
    echo "Usage: $0 <targets_file>"
    echo "Create a file with one IP/hostname per line"
    exit 1
fi

echo "=== Multi-Target FTP Scanner ==="
echo "Targets file: $TARGETS_FILE"
echo

while read target; do
    if [ ! -z "$target" ]; then
        echo "=== Scanning $target ==="
        
        # Quick banner grab
        banner=$(timeout 3 nc $target 21 2>/dev/null | head -1)
        if [ ! -z "$banner" ]; then
            echo "✓ FTP service detected: $banner"
            
            # Quick anonymous test
            anon_result=$(echo -e "user anonymous anonymous\nquit" | timeout 5 ftp -n $target 2>/dev/null)
            if echo "$anon_result" | grep -q "230 Login successful"; then
                echo "  ✓ Anonymous access enabled"
                
                # Quick file listing
                file_count=$(curl -s --connect-timeout 5 ftp://$target/ 2>/dev/null | wc -l)
                echo "  Files/directories found: $file_count"
            else
                echo "  ✗ Anonymous access disabled"
            fi
        else
            echo "✗ No FTP service detected"
        fi
        echo
    fi
done < $TARGETS_FILE
```

---

## 🔍 6. Output Analysis and Interpretation

### FTP Response Code Analysis:

| Code | Meaning | Implication |
|------|---------|-------------|
| `220` | Service ready | FTP server is running |
| `230` | User logged in | Authentication successful |
| `331` | Username OK, need password | Valid username |
| `530` | Login incorrect | Authentication failed |
| `425` | Can't open data connection | Firewall/NAT issues |
| `150` | File status okay | Transfer starting |
| `226` | Transfer complete | File transfer successful |
| `550` | File unavailable | File not found or no permission |

### Banner Analysis Examples:

**vsFTPd Server:**
```bash
220 (vsFTPd 3.0.3)
```
- **Software**: Very Secure FTP Daemon
- **Version**: 3.0.3
- **Platform**: Typically Linux
- **Security**: Generally secure, check for version-specific CVEs

**ProFTPD Server:**
```bash
220 ProFTPD 1.3.5 Server (FTP Server) [192.168.1.100]
```
- **Software**: ProFTPD
- **Version**: 1.3.5
- **Platform**: Unix/Linux
- **Note**: Shows IP address in banner

**FileZilla Server:**
```bash
220-FileZilla Server 0.9.60 beta
220-written by Tim Kosse (tim.kosse@filezilla-project.org)
220 Please visit https://filezilla-project.org/
```
- **Software**: FileZilla Server
- **Version**: 0.9.60 beta
- **Platform**: Windows
- **Note**: Multi-line banner with contact info

### Directory Listing Analysis:

**Unix-style listing:**
```bash
drwxr-xr-x    2 0        0            4096 Sep 16 12:00 documents
-rw-r--r--    1 1000     1000          156 Sep 16 12:00 readme.txt
lrwxrwxrwx    1 0        0               7 Sep 16 12:00 link -> target
```

**Analysis:**
- `d` = directory, `-` = file, `l` = symbolic link
- Permission bits show read/write/execute for owner/group/other
- User/Group IDs (0 = root, 1000 = first regular user)
- File sizes and timestamps

### Security Configuration Analysis:

**Anonymous Access Patterns:**
```bash
# Full anonymous access
230 Login successful.

# Anonymous with restricted access
230 Anonymous access granted, restrictions apply.

# Anonymous disabled
530 Permission denied.
```

**Common Writable Directories:**
- `/incoming` - Often writable for uploads
- `/pub` - Public directory, may be writable
- `/tmp` - Temporary directory
- `/upload` - Dedicated upload directory

---

## 🔗 7. Integration with Other Tools

### With Nmap:
```bash
# FTP service detection
nmap -p 21 --script ftp-anon,ftp-syst <target>

# Follow up with CLI tools
nmap -p 21 --script ftp-anon <target> | grep -q "Anonymous FTP login allowed" && echo "Testing with CLI tools..."
```

### With Hydra (Brute Force):
```bash
# After banner analysis, attempt brute force
hydra -l admin -P passwords.txt ftp://<target>

# Use discovered usernames
hydra -L usernames.txt -P passwords.txt ftp://<target>
```

### With Metasploit:
```bash
# Use auxiliary modules after CLI enumeration
use auxiliary/scanner/ftp/anonymous
set RHOSTS <target>
run

# Validate results with CLI
curl ftp://<target>/
```

### Pipeline Integration:
```bash
#!/bin/bash
# FTP enumeration to exploitation pipeline

TARGET=$1

echo "=== FTP Enumeration Pipeline ==="

# Step 1: Service detection
if nc -zv $TARGET 21 2>/dev/null; then
    echo "✓ FTP service detected"
    
    # Step 2: Banner grab
    banner=$(timeout 3 nc $TARGET 21 2>/dev/null | head -1)
    echo "Banner: $banner"
    
    # Step 3: Anonymous test
    if echo -e "user anonymous anonymous\nquit" | ftp -n $TARGET 2>/dev/null | grep -q "230"; then
        echo "✓ Anonymous access available"
        
        # Step 4: Enumerate with multiple tools
        echo "Enumerating with curl..."
        curl ftp://$TARGET/
        
        echo "Enumerating with Nmap..."
        nmap -p 21 --script ftp-anon,ftp-syst $TARGET
        
        # Step 5: Download interesting files
        echo "Downloading files..."
        wget -r --no-parent ftp://anonymous:@$TARGET/
        
    else
        echo "✗ Anonymous access denied, trying brute force..."
        hydra -l ftp -P /usr/share/wordlists/rockyou.txt ftp://$TARGET
    fi
else
    echo "✗ FTP service not available"
fi
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Refused** | `Connection refused` or timeout | Check if FTP service is running |
| **Connection Hangs** | Commands don't respond | Try passive mode with `-p` flag |
| **Login Fails** | `530 Login incorrect` | Verify credentials or try anonymous |
| **Transfer Fails** | `425 Can't open data connection` | Firewall blocking data ports |
| **Binary Corruption** | Downloaded files corrupted | Use binary mode for non-text files |

### Debugging Commands:
```bash
# Test FTP port
telnet <target> 21

# Test with different clients
curl -v ftp://<target>/
ftp -d <target>  # Debug mode

# Check passive vs active mode
ftp> passive
ftp> !passive

# Network troubleshooting
traceroute <target>
nmap -p 21 <target>
```

### Common Error Scenarios:

**Scenario 1: Passive Mode Issues**
```bash
ftp> ls
200 PORT command successful. Consider using PASV.
425 Failed to establish connection.
```
**Solution**: Enable passive mode with `passive` command

**Scenario 2: Binary vs ASCII Mode**
```bash
# File downloaded but corrupted
```
**Solution**: Use `binary` mode for executable files, `ascii` for text

**Scenario 3: Firewall Blocking Data Connection**
```bash
ftp> ls
200 PORT command successful.
# Hangs indefinitely
```
**Solution**: Configure firewall to allow FTP data ports (20) or use passive mode

### Performance Optimization:
```bash
# Faster connections with timeout
timeout 30 ftp <target>

# Parallel downloads with lftp
lftp -e "pget -n 4 largefile.zip" ftp://target/

# Resume interrupted downloads
wget -c ftp://target/largefile.zip
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only test FTP services you own or have permission to test
- **Data Sensitivity**: FTP often contains sensitive business data
- **Privacy Respect**: Don't access personal or confidential files unnecessarily
- **Responsible Disclosure**: Report vulnerabilities through appropriate channels

### Operational Security:
- **Logging Impact**: FTP connections are logged extensively
- **Network Detection**: FTP traffic is easily monitored
- **Data Transfer Logs**: All file transfers are typically logged
- **Timing**: Space out enumeration to avoid detection patterns

### Detection Indicators:
FTP enumeration may be detected through:
- **FTP Server Logs**: Connection attempts, login failures
- **Network Monitoring**: FTP traffic analysis
- **Failed Authentication**: Multiple anonymous login attempts
- **Directory Traversal**: Unusual directory access patterns

### Defensive Countermeasures:
Organizations can protect against FTP enumeration by:
- **Disable Anonymous Access**: Require authentication for all access
- **Strong Authentication**: Implement complex passwords and account lockout
- **Network Segmentation**: Restrict FTP access to necessary networks
- **Monitoring**: Deploy FTP access monitoring and alerting
- **Encryption**: Use FTPS or SFTP instead of plain FTP

### Risk Assessment:
| Activity | Detection Risk | Impact | Data Sensitivity |
|----------|----------------|--------|------------------|
| Banner Grabbing | 🟢 Low | 🟢 Low | Version disclosure |
| Anonymous Testing | 🟡 Medium | 🟠 High | Unauthorized access |
| File Enumeration | 🟠 High | 🟠 High | Information disclosure |
| File Download | 🔴 High | 🔴 High | Data exfiltration |

---

## 📖 References

### Official Documentation:
- [RFC 959 - FTP Protocol](https://tools.ietf.org/html/rfc959) - Official FTP protocol specification
- [vsFTPd Documentation](https://security.appspot.com/vsftpd.html) - Popular FTP server documentation
- [ProFTPD Documentation](http://www.proftpd.org/docs/) - ProFTPD server reference

### Security Resources:
- [FTP Security Best Practices](https://www.sans.org/reading-room/whitepapers/protocols/ftp-security-1327) - SANS security guide
- [OWASP FTP Security](https://owasp.org/www-community/vulnerabilities/Insecure_Transport) - Web application security guidance
- [FTP Vulnerabilities Database](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=ftp) - CVE listings

### Learning Resources:
- [FTP Protocol Analysis](https://www.wireshark.org/docs/dfref/f/ftp.html) - Wireshark protocol reference
- [Linux FTP Administration](https://tldp.org/HOWTO/FTP.html) - Linux Documentation Project
- [Network Penetration Testing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp) - Practical testing methodology

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Banner grab
nc <target> 21

# Anonymous test
ftp <target>  # user: anonymous, pass: anonymous

# Directory listing
curl ftp://<target>/

# File download
wget ftp://anonymous:@<target>/file.txt
```

### Key FTP Commands:
```bash
# Inside FTP client
ls                     # List directory
cd directory           # Change directory
get filename           # Download file
put filename           # Upload file
binary                 # Binary mode
passive               # Passive mode
```

### Response Codes:
- **220** - Service ready
- **230** - Login successful  
- **331** - Need password
- **530** - Login failed
- **150/226** - Transfer successful

### File Types of Interest:
- **Configuration**: `.conf`, `.ini`, `.cfg`
- **Logs**: `.log`, `.txt`
- **Backups**: `.bak`, `.backup`, `.zip`
- **Databases**: `.db`, `.sql`, `.sqlite`

### Success Indicators:
- Banner retrieved successfully
- Anonymous login accepted
- Directory listing obtained
- Files accessible for download

### Time Estimates:
- **Banner grab**: 5-10 seconds
- **Anonymous test**: 10-30 seconds  
- **Directory enumeration**: 1-5 minutes
- **File download**: Varies by size

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting FTP enumeration and file access.*
