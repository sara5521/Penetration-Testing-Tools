# 📂 smbclient - SMB Share Client Tool

This section covers the smbclient tool for accessing and interacting with SMB shares on Windows and Samba servers.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Tool Overview](#-tool-overview)
- [Prerequisites](#-prerequisites)
- [Share Discovery](#-1-share-discovery)
- [Anonymous Access Testing](#-2-anonymous-access-testing)
- [Authenticated Access](#-3-authenticated-access)
- [Interactive File Operations](#-4-interactive-file-operations)
- [Automated Operations](#-5-automated-operations)
- [Advanced Techniques](#-6-advanced-techniques)
- [Output Analysis](#-7-output-analysis-and-interpretation)
- [Integration with Other Tools](#-8-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

The `smbclient` tool from the Samba suite helps you:
- List available SMB shares on target systems
- Test for anonymous (null session) access
- Access and browse SMB shared folders
- Upload and download files from shares
- Execute commands on accessible shares
- Validate SMB security configurations
- Perform file-based reconnaissance and data exfiltration

---

## 🗂️ Tool Overview

| Attribute | Details |
|-----------|---------|
| **Tool Name** | smbclient |
| **Package** | samba-clients / samba |
| **Protocol** | SMB/CIFS |
| **Transport** | TCP |
| **Default Port** | 445/tcp |
| **Authentication** | Optional (supports anonymous) |
| **Platform** | Linux/Unix |
| **Interface** | Command-line and interactive |

---

## 📋 Prerequisites

### Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install smbclient

# CentOS/RHEL
sudo yum install samba-client

# Or full Samba suite
sudo apt install samba
```

### Network Requirements:
- SMB service running on target (port 445)
- Network connectivity to target
- Appropriate permissions for share access

### Quick Service Check:
```bash
# Test SMB service availability
nmap -p 445 <target-ip>

# Quick SMB banner grab
echo "exit" | smbclient -L <target-ip> -N
```

---

## 🔧 smbclient SMB Operations

## 📋 1. Share Discovery

### Basic Share Listing:
```bash
smbclient -L <target-ip>
```

### Anonymous Share Listing:
```bash
smbclient -L <target-ip> -N
```

**📌 INE Lab Example:**
```bash
smbclient -L demo.ine.local -N
```

**📸 Sample Output:**
```bash
Sharename       Type      Comment
---------       ----      -------
public          Disk      
john            Disk      
aisha           Disk      
emma            Disk      
everyone        Disk      
IPC$            IPC       IPC Service (samba.recon.lab)

Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------
        SAMBA-RECON         

        Workgroup            Master
        ---------            -------
        RECONLABS           SAMBA-RECON
```

### Command Options for Share Listing:
| Option | Description | Example |
|--------|-------------|---------|
| `-L` | List shares on server | `smbclient -L 192.168.1.100` |
| `-N` | No password (anonymous) | `smbclient -L 192.168.1.100 -N` |
| `-U` | Specify username | `smbclient -L 192.168.1.100 -U administrator` |
| `-W` | Specify workgroup/domain | `smbclient -L 192.168.1.100 -W CORP` |
| `-I` | Specify IP address | `smbclient -L hostname -I 192.168.1.100` |
| `-p` | Specify port | `smbclient -L 192.168.1.100 -p 445` |

---

## 🔓 2. Anonymous Access Testing

### Test Anonymous Share Access:
```bash
smbclient //<target-ip>/IPC$ -N
smbclient //<target-ip>/public -N
```

### Systematic Anonymous Testing:
```bash
#!/bin/bash
# Test anonymous access to discovered shares

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Testing Anonymous Access to $TARGET ==="

# Get share list
shares=$(smbclient -L $TARGET -N 2>/dev/null | grep "Disk\|IPC" | awk '{print $1}')

for share in $shares; do
    echo "Testing share: $share"
    result=$(echo "dir" | smbclient "//$TARGET/$share" -N 2>/dev/null)
    
    if [[ $? -eq 0 ]]; then
        echo "✓ Anonymous access successful to $share"
        echo "Contents:"
        echo "$result" | head -10
        echo
    else
        echo "✗ Anonymous access denied to $share"
    fi
done
```

**📸 Anonymous Access Success:**
```bash
smbclient //demo.ine.local/public -N
Try "help" to get a list of possible commands.
smb: \>
```

**📸 Anonymous Access Denied:**
```bash
smbclient //demo.ine.local/private -N
session setup failed: NT_STATUS_ACCESS_DENIED
```

---

## 🔐 3. Authenticated Access

### Using Username and Password:
```bash
smbclient //<target-ip>/<share> -U <username>
# Enter password when prompted
```

### Non-Interactive Authentication:
```bash
smbclient //<target-ip>/<share> -U <username>%<password>
```

### Domain Authentication:
```bash
smbclient //<target-ip>/<share> -U <domain>\\<username>
smbclient //<target-ip>/<share> -U <username> -W <domain>
```

**📌 Examples:**
```bash
# Standard authentication
smbclient //demo.ine.local/admin$ -U administrator

# With password
smbclient //demo.ine.local/C$ -U administrator%password123

# Domain user
smbclient //demo.ine.local/shared -U CORP\\john.doe
```

### Kerberos Authentication:
```bash
# Using Kerberos ticket
smbclient //server.domain.com/share -k

# Specify principal
smbclient //server.domain.com/share -U user@DOMAIN.COM -k
```

---

## 💻 4. Interactive File Operations

Once connected to a share, use these interactive commands:

### Navigation Commands:
```bash
smb: \> dir                 # List directory contents
smb: \> ls                  # Alternative to dir
smb: \> cd folder           # Change directory
smb: \> pwd                 # Show current directory
smb: \> lcd /local/path     # Change local directory
```

### File Transfer Commands:
```bash
smb: \> get filename.txt           # Download file
smb: \> mget *.txt                 # Download multiple files
smb: \> put local_file.txt         # Upload file
smb: \> mput *.doc                 # Upload multiple files
```

### File Management Commands:
```bash
smb: \> mkdir newfolder            # Create directory
smb: \> rmdir emptyfolder          # Remove directory
smb: \> del filename.txt           # Delete file
smb: \> rename old.txt new.txt     # Rename file
```

### Information Commands:
```bash
smb: \> help                       # Show available commands
smb: \> !ls                        # Execute local shell command
smb: \> exit                       # Exit smbclient
smb: \> quit                       # Alternative to exit
```

**📸 Sample Interactive Session:**
```bash
smbclient //demo.ine.local/public -N
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Sep 16 14:30:00 2025
  ..                                  D        0  Mon Sep 16 14:25:00 2025
  document.txt                        A      156  Mon Sep 16 14:30:00 2025
  backup.zip                          A     2048  Mon Sep 16 14:25:00 2025

                50331648 blocks of size 1024. 47185920 blocks available

smb: \> get document.txt
getting file \document.txt of size 156 as document.txt (0.1 KiloBytes/sec)

smb: \> !ls -la document.txt
-rw-r--r-- 1 user user 156 Sep 16 14:31 document.txt

smb: \> exit
```

---

## ⚡ 5. Automated Operations

### Non-Interactive File Download:
```bash
# Download single file
smbclient //<target>/<share> -N -c "get filename.txt; quit"

# Download all files
smbclient //<target>/<share> -N -c "prompt; recurse; mget *; quit"
```

### Batch Command Execution:
```bash
# Create command file
echo -e "dir\nget important.txt\nmget *.log\nquit" > smb_commands.txt

# Execute batch commands
smbclient //<target>/<share> -N < smb_commands.txt
```

### Automated Share Enumeration:
```bash
#!/bin/bash
# Comprehensive share enumeration and file listing

TARGET=$1
OUTPUT_DIR="smb_enumeration"
mkdir -p $OUTPUT_DIR

echo "=== SMB Share Enumeration of $TARGET ==="

# Get share list
echo "Discovering shares..."
smbclient -L $TARGET -N > $OUTPUT_DIR/shares.txt 2>&1

# Extract share names
shares=$(grep -E "^\s+\w+\s+(Disk|IPC)" $OUTPUT_DIR/shares.txt | awk '{print $1}')

for share in $shares; do
    echo "Enumerating share: $share"
    share_file="$OUTPUT_DIR/share_$share.txt"
    
    # Test access and list contents
    smbclient "//$TARGET/$share" -N -c "dir; quit" > $share_file 2>&1
    
    if grep -q "NT_STATUS" $share_file; then
        echo "✗ Access denied to $share"
    else
        echo "✓ Successfully accessed $share"
        
        # Download small text files for analysis
        smbclient "//$TARGET/$share" -N -c "prompt; mget *.txt *.log *.conf; quit" 2>/dev/null
    fi
done

echo "Enumeration complete. Results in $OUTPUT_DIR/"
```

### Recursive File Download:
```bash
#!/bin/bash
# Recursively download all accessible files

TARGET=$1
SHARE=$2

if [ -z "$TARGET" ] || [ -z "$SHARE" ]; then
    echo "Usage: $0 <target> <share>"
    exit 1
fi

echo "=== Recursive Download from //$TARGET/$SHARE ==="

# Create download directory
mkdir -p "downloads_${TARGET}_${SHARE}"
cd "downloads_${TARGET}_${SHARE}"

# Download everything
smbclient "//$TARGET/$SHARE" -N -c "
lcd $(pwd)
prompt
recurse
mget *
quit
"

echo "Download complete in: $(pwd)"
find . -type f | head -20
```

---

## 🚀 6. Advanced Techniques

### Share Mounting (Alternative Approach):
```bash
# Mount SMB share to local filesystem
sudo mount -t cifs //<target>/<share> /mnt/smb -o username=guest,password=

# Browse mounted share
ls -la /mnt/smb/

# Unmount when done
sudo umount /mnt/smb
```

### Protocol Version Testing:
```bash
# Force SMBv1
smbclient //<target>/<share> -N --option='client min protocol=NT1' --option='client max protocol=NT1'

# Force SMBv2
smbclient //<target>/<share> -N --option='client min protocol=SMB2' --option='client max protocol=SMB2'

# Force SMBv3
smbclient //<target>/<share> -N --option='client min protocol=SMB3'
```

### Large File Handling:
```bash
# Resume interrupted downloads
smbclient //<target>/<share> -N -c "get largefile.zip largefile.zip.partial; quit"

# Download with progress
smbclient //<target>/<share> -N -c "progress; get largefile.zip; quit"
```

### Share Scanning Across Networks:
```bash
#!/bin/bash
# Network-wide SMB share discovery

NETWORK=$1  # e.g., 192.168.1.0/24
if [ -z "$NETWORK" ]; then
    echo "Usage: $0 <network>"
    exit 1
fi

echo "=== Network SMB Share Discovery ==="

# Discover SMB hosts
nmap -p 445 --open $NETWORK | grep "Nmap scan report" | awk '{print $5}' | while read host; do
    echo "=== Scanning $host ==="
    
    # List shares
    shares=$(timeout 10 smbclient -L $host -N 2>/dev/null | grep "Disk" | awk '{print $1}')
    
    if [[ ! -z "$shares" ]]; then
        echo "Found shares on $host:"
        echo "$shares"
        
        # Test anonymous access to each share
        for share in $shares; do
            result=$(timeout 5 smbclient "//$host/$share" -N -c "dir; quit" 2>/dev/null)
            if [[ $? -eq 0 ]]; then
                echo "  ✓ Anonymous access to $host/$share"
            fi
        done
    fi
    echo
done
```

---

## 🔍 7. Output Analysis and Interpretation

### Share Type Analysis:

| Share Type | Description | Security Implication |
|------------|-------------|---------------------|
| `Disk` | Regular file share | 🟠 Potential data access |
| `IPC` | Inter-Process Communication | 🟡 System enumeration |
| `Printer` | Shared printer | 🟢 Limited security risk |
| `Unknown` | Undefined share type | 🟡 Requires investigation |

### Common Share Names and Their Significance:

| Share Name | Typical Contents | Risk Level |
|------------|------------------|------------|
| `C$` | Root C: drive | 🔴 Critical - Full system access |
| `ADMIN$` | Windows directory | 🔴 Critical - Administrative access |
| `IPC$` | Named pipes | 🟡 Medium - Enumeration vector |
| `public` | Public documents | 🟡 Medium - Information disclosure |
| `users` | User home directories | 🟠 High - Personal data |
| `backup` | Backup files | 🟠 High - Sensitive data |
| `shared` | Shared documents | 🟡 Medium - Business data |

### File Type Analysis Priority:

**High Priority Files:**
- Configuration files (`.conf`, `.ini`, `.xml`)
- Database files (`.db`, `.sqlite`, `.mdb`)
- Backup files (`.bak`, `.backup`, `.zip`)
- Key files (`.key`, `.pem`, `.p12`)
- Password files (`password`, `passwd`, `.pwd`)

**Medium Priority Files:**
- Log files (`.log`, `.txt`)
- Document files (`.doc`, `.pdf`, `.xls`)
- Script files (`.bat`, `.ps1`, `.sh`)
- Source code (`.py`, `.java`, `.c`)

### Access Pattern Analysis:
```bash
# Sample directory listing analysis
  .                                   D        0  Mon Sep 16 14:30:00 2025
  ..                                  D        0  Mon Sep 16 14:25:00 2025
  confidential.txt                    A      156  Mon Sep 16 14:30:00 2025
  database_backup.sql                 A    50248  Mon Sep 16 14:25:00 2025
  passwords.xlsx                      A     2048  Mon Sep 16 14:20:00 2025
```

**Analysis:**
- `D` = Directory, `A` = Archive (file)
- File sizes indicate content volume
- Timestamps show recent activity
- Filenames suggest sensitive data

---

## 🔗 8. Integration with Other Tools

### With enum4linux:
```bash
# enum4linux share enumeration
enum4linux -S <target>

# Verify with smbclient
smbclient -L <target> -N
```

### With Nmap:
```bash
# Nmap SMB share enumeration
nmap -p 445 --script smb-enum-shares <target>

# Follow up with smbclient access test
nmap --script smb-enum-shares <target> | grep -o "\\\\.*\\\\.*" | while read share; do
    echo "Testing $share"
    smbclient "$share" -N -c "dir; quit"
done
```

### With CrackMapExec:
```bash
# CrackMapExec share enumeration
crackmapexec smb <target> --shares

# Use smbclient for detailed file access
crackmapexec smb <target> --shares | grep READ | while read line; do
    share=$(echo $line | awk '{print $1}')
    smbclient "//<target>/$share" -N -c "recurse; ls; quit"
done
```

### Data Processing Pipeline:
```bash
#!/bin/bash
# SMB to data analysis pipeline

TARGET=$1
OUTPUT_DIR="smb_analysis"
mkdir -p $OUTPUT_DIR/{shares,downloads,analysis}

echo "=== SMB Analysis Pipeline for $TARGET ==="

# Step 1: Discover shares
echo "Step 1: Share discovery"
smbclient -L $TARGET -N > $OUTPUT_DIR/shares/share_list.txt

# Step 2: Test anonymous access
echo "Step 2: Anonymous access testing"
grep "Disk" $OUTPUT_DIR/shares/share_list.txt | awk '{print $1}' | while read share; do
    echo "Testing $share..."
    smbclient "//$TARGET/$share" -N -c "dir; quit" > $OUTPUT_DIR/shares/access_$share.txt 2>&1
    
    if ! grep -q "NT_STATUS" $OUTPUT_DIR/shares/access_$share.txt; then
        echo "✓ Accessible: $share"
        
        # Step 3: Download interesting files
        echo "Step 3: Downloading files from $share"
        cd $OUTPUT_DIR/downloads
        smbclient "//$TARGET/$share" -N -c "
        prompt
        mget *.txt *.log *.conf *.ini *.xml *.sql *.bak
        quit
        " 2>/dev/null
        cd - >/dev/null
    fi
done

# Step 4: Analyze downloaded files
echo "Step 4: File analysis"
find $OUTPUT_DIR/downloads -type f | while read file; do
    echo "=== Analysis of $(basename $file) ===" >> $OUTPUT_DIR/analysis/file_analysis.txt
    file "$file" >> $OUTPUT_DIR/analysis/file_analysis.txt
    
    # Look for sensitive patterns
    grep -i -E "(password|secret|key|token|credential)" "$file" >> $OUTPUT_DIR/analysis/sensitive_data.txt 2>/dev/null
done

echo "Analysis complete. Results in $OUTPUT_DIR/"
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Error Message | Solution |
|-------|---------------|----------|
| **Access Denied** | `NT_STATUS_ACCESS_DENIED` | Share requires authentication |
| **Bad Network Name** | `NT_STATUS_BAD_NETWORK_NAME` | Share name doesn't exist |
| **Connection Refused** | `Connection to <target> failed` | SMB service not running |
| **Protocol Error** | `protocol negotiation failed` | SMB version mismatch |
| **Tool Not Found** | `command not found` | Install samba-clients package |

### Debugging Commands:
```bash
# Test basic connectivity
telnet <target> 445

# Verbose smbclient output
smbclient -L <target> -N -d 3

# Check SMB protocol support
smbclient -L <target> -N --option='client min protocol=NT1'

# Alternative share listing
enum4linux -S <target>
```

### Common Error Scenarios:

**Scenario 1: Access Denied**
```bash
smbclient //192.168.1.100/admin$ -N
session setup failed: NT_STATUS_ACCESS_DENIED
```
**Solution**: Share requires authentication, try with valid credentials

**Scenario 2: Bad Network Name**
```bash
smbclient //192.168.1.100/nonexistent -N
tree connect failed: NT_STATUS_BAD_NETWORK_NAME
```
**Solution**: Verify share name with `smbclient -L`

**Scenario 3: Protocol Negotiation Failed**
```bash
smbclient //192.168.1.100/share -N
protocol negotiation failed: NT_STATUS_INVALID_NETWORK_RESPONSE
```
**Solution**: Try different SMB protocol versions

### Performance Optimization:
```bash
# Faster operations with timeout
timeout 30 smbclient -L <target> -N

# Parallel share testing
for share in share1 share2 share3; do
    timeout 10 smbclient "//<target>/$share" -N -c "dir; quit" &
done
wait
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only access shares you have permission to test
- **Data Handling**: Treat downloaded data with appropriate security measures
- **Scope Compliance**: Stay within authorized testing boundaries
- **Privacy Respect**: Don't access personal or confidential data unnecessarily

### Operational Security:
- **Logging Impact**: SMB access generates detailed Windows security logs
- **File Access Logs**: File downloads are logged in detail
- **Network Detection**: Large file transfers may trigger monitoring
- **Timing**: Space out operations to avoid detection patterns

### Detection Indicators:
SMB client activities may be detected through:
- **Windows Event Logs**: Logon events (4624), file access (4656, 4658)
- **Network Monitoring**: SMB traffic volume and patterns
- **File Access Logs**: Audit logs for file downloads
- **IDS Signatures**: Unusual SMB enumeration patterns

### Defensive Countermeasures:
Organizations can protect against SMB client attacks by:
- **Access Control**: Implement proper share permissions
- **Network Segmentation**: Limit SMB access between network segments
- **Monitoring**: Deploy SMB access monitoring and alerting
- **Data Classification**: Protect sensitive data with encryption
- **Audit Logging**: Enable comprehensive SMB audit logging

### Risk Assessment:
| Activity | Detection Risk | Impact | Data Sensitivity |
|----------|----------------|--------|------------------|
| Share Listing | 🟡 Medium | 🟡 Medium | Network mapping |
| Anonymous Access | 🟠 High | 🟠 High | Unauthorized access |
| File Download | 🔴 High | 🔴 High | Data exfiltration |
| File Upload | 🔴 High | 🔴 High | Potential malware |

---

## 📖 References

### Official Documentation:
- [Samba smbclient Manual](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) - Official tool documentation
- [SMB Protocol Guide](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) - Microsoft SMB reference
- [CIFS Technical Reference](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/) - Detailed protocol specifications

### Security Research:
- [SMB Share Security](https://www.sans.org/reading-room/whitepapers/protocols/smb-security-best-practices-1109) - SANS best practices
- [SMB Vulnerabilities Database](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=smb) - CVE listings
- [Windows File Sharing Security](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/) - Microsoft security guide

### Learning Resources:
- [Samba Administration Guide](https://www.samba.org/samba/docs/current/Samba-HOWTO-Collection/) - Comprehensive Samba documentation
- [SMB Penetration Testing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) - Practical testing methodology
- [Windows SMB Hardening](https://www.cisecurity.org/controls/) - CIS security controls

---

## 📋 Quick Reference

### Essential Commands:
```bash
# List shares anonymously
smbclient -L <target> -N

# Connect to share
smbclient //<target>/<share> -N

# Batch file download
smbclient //<target>/<share> -N -c "prompt; mget *.txt; quit"

# Authenticated access
smbclient //<target>/<share> -U username%password
```

### Key Interactive Commands:
```bash
# Inside smbclient shell
dir                    # List contents
get filename           # Download file
put filename           # Upload file
cd directory           # Change directory
help                   # Show commands
```

### Share Priority Assessment:
1. **Critical**: `C$`, `ADMIN$`, `backup`
2. **High**: User directories, `shared`, `public`
3. **Medium**: `IPC$`, printer shares
4. **Low**: Empty or inaccessible shares

### File Types of Interest:
- **Configuration**: `.conf`, `.ini`, `.xml`, `.json`
- **Credentials**: `password*`, `*.key`, `*.pem`
- **Databases**: `.db`, `.sqlite`, `.mdb`
- **Backups**: `.bak`, `.backup`, `.zip`

### Success Indicators:
- Share listing obtained without credentials
- Interactive `smb: \>` prompt achieved
- Files successfully downloaded
- Write access confirmed (if testing)

### Time Estimates:
- **Share discovery**: 10-30 seconds
- **Anonymous testing**: 1-3 minutes
- **File enumeration**: 5-15 minutes per share
- **Data download**: Varies by file size

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before accessing SMB shares and downloading data.*
