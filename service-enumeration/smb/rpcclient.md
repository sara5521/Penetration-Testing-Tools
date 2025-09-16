# 🛠️ rpcclient - SMB RPC Enumeration Tool

This section covers the rpcclient tool for interacting with Windows RPC services over SMB to enumerate system information.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Tool Overview](#-tool-overview)
- [Prerequisites](#-prerequisites)
- [Anonymous Connection Testing](#-1-anonymous-connection-testing)
- [Authenticated Access](#-2-authenticated-access)
- [RPC Enumeration Commands](#-3-rpc-enumeration-commands)
- [Advanced Techniques](#-4-advanced-techniques)
- [Output Analysis](#-5-output-analysis-and-interpretation)
- [Integration with Other Tools](#-6-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

The `rpcclient` tool from the Samba suite helps you:
- Test for anonymous SMB/RPC access (null sessions)
- Interact with Windows RPC (Remote Procedure Call) services
- Enumerate users, groups, and domain information
- Query system information and shares
- Perform domain reconnaissance without credentials
- Validate SMB security configurations

---

## 🗂️ Tool Overview

| Attribute | Details |
|-----------|---------|
| **Tool Name** | rpcclient |
| **Package** | samba-clients / samba |
| **Protocol** | SMB/CIFS with RPC |
| **Transport** | TCP |
| **Default Port** | 445/tcp (SMB), 135/tcp (RPC) |
| **Authentication** | Optional (supports null sessions) |
| **Platform** | Linux/Unix |

---

## 📋 Prerequisites

### Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install samba-clients

# CentOS/RHEL
sudo yum install samba-client

# Or full Samba suite
sudo apt install samba
```

### Network Requirements:
- SMB service running on target (port 445)
- RPC services enabled
- Network connectivity to target

### Quick Service Check:
```bash
# Test SMB connectivity
nmap -p 445 <target-ip>

# Check RPC services
nmap -p 135,445 <target-ip>
```

---

## 🔧 rpcclient RPC Enumeration

## 🔓 1. Anonymous Connection Testing

### Basic Null Session Test:
```bash
rpcclient -U "" -N <target-ip>
```

### Command Options:
| Option | Description | Purpose |
|--------|-------------|---------|
| `-U ""` | Empty username | Attempts null session |
| `-N` | No password prompt | Skip password authentication |
| `-W <domain>` | Specify workgroup/domain | Domain-specific connection |
| `-I <ip>` | Specify IP address | Direct IP connection |

**📌 INE Lab Example:**
```bash
rpcclient -U "" -N demo.ine.local
```

**📸 Success Indicator:**
```bash
rpcclient $>
```

**🟢 Interpretation:**
- Successful connection established
- Null session allowed on target
- Interactive RPC shell available
- No authentication required

**❌ Failure Examples:**
```bash
# Access denied
Cannot connect to server. Error was NT_STATUS_ACCESS_DENIED

# Connection refused
Cannot connect to server. Error was NT_STATUS_CONNECTION_REFUSED

# Logon failure
Cannot connect to server. Error was NT_STATUS_LOGON_FAILURE
```

### Alternative Anonymous Connection Methods:
```bash
# Using guest account
rpcclient -U "guest" -N <target-ip>

# Specifying domain
rpcclient -U "" -N -W WORKGROUP <target-ip>

# Using IP directly
rpcclient -U "" -N -I <target-ip>
```

---

## 🔐 2. Authenticated Access

### Using Valid Credentials:
```bash
rpcclient -U <username> <target-ip>
# Enter password when prompted
```

### Non-Interactive Authentication:
```bash
rpcclient -U <username>%<password> <target-ip>
```

### Domain Authentication:
```bash
rpcclient -U <domain>\\<username> <target-ip>
rpcclient -U <username> -W <domain> <target-ip>
```

**📌 Examples:**
```bash
# Standard authentication
rpcclient -U administrator demo.ine.local

# With password
rpcclient -U administrator%password123 demo.ine.local

# Domain user
rpcclient -U CORP\\john.doe demo.ine.local
```

---

## 📊 3. RPC Enumeration Commands

Once connected to the rpcclient shell, use these commands:

### User Enumeration:
```bash
# List all domain users
enumdomusers

# Get user information
queryuser <RID>
queryuser 500  # Administrator
queryuser 501  # Guest

# Lookup user by name
lookupnames administrator
lookupnames guest

# Get user groups
queryusergroups <RID>
```

**📸 Sample `enumdomusers` Output:**
```bash
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[bob] rid:[0x3f2]
user:[alice] rid:[0x3f3]
```

### Group Enumeration:
```bash
# List all domain groups
enumdomgroups

# Get group information
querygroup <RID>
querygroup 512  # Domain Admins
querygroup 513  # Domain Users

# List group members
querygroupmem <RID>
```

**📸 Sample `enumdomgroups` Output:**
```bash
rpcclient $> enumdomgroups
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
```

### System Information:
```bash
# Server information
srvinfo

# Domain information
querydominfo

# Get domain SID
lsaquery

# List privileges
enumprivs
```

**📸 Sample `srvinfo` Output:**
```bash
rpcclient $> srvinfo
        SAMBA-RECON    Wk Sv PrQ Unx NT SNT samba-recon
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
```

### Share Enumeration:
```bash
# List all shares
netshareenumall

# Get share information
netsharegetinfo <share_name>
netsharegetinfo C$
```

**📸 Sample `netshareenumall` Output:**
```bash
rpcclient $> netshareenumall
netname: IPC$
        remark: Remote IPC
        path:   C:\tmp
        password:
netname: public
        remark: 
        path:   C:\home\public
        password:
```

### Security Information:
```bash
# List security accounts
lsaenumsid

# Get account policy
getdompwinfo

# List user rights
enumalsgroups domain
```

---

## 🚀 4. Advanced Techniques

### Automated Enumeration Script:
```bash
#!/bin/bash
# RPC enumeration automation script

TARGET=$1
if [ -z "$TARGET" ]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== RPC Enumeration of $TARGET ==="
echo "Date: $(date)"
echo

# Test anonymous connection
echo "Testing anonymous access..."
rpcclient -U "" -N $TARGET -c "srvinfo; quit" > /tmp/rpc_test.out 2>&1

if grep -q "platform_id" /tmp/rpc_test.out; then
    echo "✓ Anonymous access successful"
    
    echo
    echo "=== System Information ==="
    rpcclient -U "" -N $TARGET -c "srvinfo; querydominfo; quit"
    
    echo
    echo "=== User Enumeration ==="
    rpcclient -U "" -N $TARGET -c "enumdomusers; quit"
    
    echo
    echo "=== Group Enumeration ==="
    rpcclient -U "" -N $TARGET -c "enumdomgroups; quit"
    
    echo
    echo "=== Share Enumeration ==="
    rpcclient -U "" -N $TARGET -c "netshareenumall; quit"
    
else
    echo "✗ Anonymous access denied"
    echo "Try with valid credentials: rpcclient -U username $TARGET"
fi

rm -f /tmp/rpc_test.out
```

### Batch Command Execution:
```bash
# Execute multiple commands without interactive shell
rpcclient -U "" -N <target> -c "enumdomusers; enumdomgroups; srvinfo; quit"

# Using command file
echo -e "enumdomusers\nenumdomgroups\nsrvinfo\nquit" > commands.txt
rpcclient -U "" -N <target> < commands.txt
```

### User Detail Enumeration:
```bash
#!/bin/bash
# Detailed user enumeration

TARGET=$1
echo "=== Detailed User Enumeration ==="

# Get user list and extract RIDs
rpcclient -U "" -N $TARGET -c "enumdomusers; quit" | while read line; do
    if [[ $line =~ rid:\[0x([0-9a-f]+)\] ]]; then
        rid=$((16#${BASH_REMATCH[1]}))
        username=$(echo $line | grep -o 'user:\[[^]]*\]' | sed 's/user:\[\(.*\)\]/\1/')
        
        echo "=== User: $username (RID: $rid) ==="
        rpcclient -U "" -N $TARGET -c "queryuser $rid; quit"
        echo
    fi
done
```

### Password Policy Enumeration:
```bash
# Get password policy information
rpcclient -U "" -N <target> -c "getdompwinfo; quit"
```

**📸 Sample Password Policy Output:**
```bash
rpcclient $> getdompwinfo
min_password_length: 7
password_properties: 0x00000001
        DOMAIN_PASSWORD_COMPLEX
```

---

## 🔍 5. Output Analysis and Interpretation

### User Information Analysis:

**RID (Relative Identifier) Significance:**
- `500` (0x1f4) → Administrator account
- `501` (0x1f5) → Guest account
- `502` → KRBTGT (Kerberos service account)
- `1000+` → Custom user accounts

### Group RID Analysis:
- `512` (0x200) → Domain Admins
- `513` (0x201) → Domain Users
- `514` (0x202) → Domain Guests
- `515` (0x203) → Domain Computers
- `516` (0x204) → Domain Controllers

### System Information Interpretation:
```bash
rpcclient $> srvinfo
        SAMBA-RECON    Wk Sv PrQ Unx NT SNT samba-recon
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
```

**Analysis:**
- **Computer Name**: SAMBA-RECON
- **Platform ID**: 500 (Windows NT family)
- **OS Version**: 6.1 (Windows 7/Server 2008 R2)
- **Server Type**: Workstation, Server, Print Queue, Unix, NT, etc.

### Domain Information Analysis:
```bash
rpcclient $> querydominfo
Domain:         WORKGROUP
Comment:        
Total Users:    3
Total Groups:   0
Total Aliases:  0
Sequence No:    1
Force Logoff:   -1
Domain Server State: 0x1
Server Role:    ROLE_DOMAIN_MEMBER
Unknown 3:      0x1
```

**Key Findings:**
- **Domain Name**: WORKGROUP (workgroup vs domain)
- **User Count**: 3 users total
- **Server Role**: Domain member vs domain controller
- **Security Implications**: Workgroup setup (less secure than domain)

---

## 🔗 6. Integration with Other Tools

### With enum4linux:
```bash
# enum4linux uses rpcclient internally
enum4linux -a <target>

# Compare with manual rpcclient enumeration
rpcclient -U "" -N <target> -c "enumdomusers; enumdomgroups; quit"
```

### With smbclient:
```bash
# Use rpcclient for enumeration, smbclient for access
rpcclient -U "" -N <target> -c "netshareenumall; quit"
smbclient -L //<target> -N
```

### With Nmap:
```bash
# Nmap SMB enumeration
nmap -p 445 --script smb-enum-users,smb-enum-groups <target>

# Verify with rpcclient
rpcclient -U "" -N <target> -c "enumdomusers; enumdomgroups; quit"
```

### Data Processing Pipeline:
```bash
#!/bin/bash
# RPC to SMB enumeration pipeline

TARGET=$1
OUTPUT_DIR="rpc_enumeration"
mkdir -p $OUTPUT_DIR

echo "=== RPC Enumeration Pipeline ==="

# Step 1: Test anonymous access
echo "Testing anonymous RPC access..."
if rpcclient -U "" -N $TARGET -c "srvinfo; quit" > $OUTPUT_DIR/rpc_test.log 2>&1; then
    echo "✓ Anonymous access successful"
    
    # Step 2: System enumeration
    echo "Gathering system information..."
    rpcclient -U "" -N $TARGET -c "srvinfo; querydominfo; getdompwinfo; quit" > $OUTPUT_DIR/system_info.txt
    
    # Step 3: User enumeration
    echo "Enumerating users..."
    rpcclient -U "" -N $TARGET -c "enumdomusers; quit" > $OUTPUT_DIR/users.txt
    
    # Step 4: Group enumeration
    echo "Enumerating groups..."
    rpcclient -U "" -N $TARGET -c "enumdomgroups; quit" > $OUTPUT_DIR/groups.txt
    
    # Step 5: Share enumeration
    echo "Enumerating shares..."
    rpcclient -U "" -N $TARGET -c "netshareenumall; quit" > $OUTPUT_DIR/shares.txt
    
    # Step 6: Extract usernames for further testing
    echo "Extracting usernames..."
    grep "user:\[" $OUTPUT_DIR/users.txt | sed 's/.*user:\[\([^]]*\)\].*/\1/' > $OUTPUT_DIR/usernames.txt
    
    # Step 7: Test shares with smbclient
    echo "Testing share access..."
    while read share; do
        share_name=$(echo $share | grep -o "netname: .*" | cut -d' ' -f2)
        if [[ ! -z "$share_name" && "$share_name" != "IPC$" ]]; then
            echo "Testing share: $share_name"
            smbclient "//$TARGET/$share_name" -N -c "ls" > $OUTPUT_DIR/share_$share_name.txt 2>&1 &
        fi
    done < $OUTPUT_DIR/shares.txt
    
    wait
    echo "Enumeration complete. Results in $OUTPUT_DIR/"
    
else
    echo "✗ Anonymous access denied"
    echo "Try with credentials: rpcclient -U username $TARGET"
fi
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Error Message | Solution |
|-------|---------------|----------|
| **Access Denied** | `NT_STATUS_ACCESS_DENIED` | Null sessions may be disabled |
| **Connection Refused** | `NT_STATUS_CONNECTION_REFUSED` | SMB service not running |
| **Logon Failure** | `NT_STATUS_LOGON_FAILURE` | Invalid credentials or account locked |
| **Network Unreachable** | Connection timeout | Check network connectivity |
| **Tool Not Found** | `command not found` | Install samba-clients package |

### Debugging Commands:
```bash
# Test SMB connectivity
telnet <target> 445

# Verbose rpcclient connection
rpcclient -U "" -N -d 3 <target>

# Check SMB version
smbclient -L <target> -N

# Alternative null session test
smbclient //<target>/IPC$ -N
```

### Common Error Scenarios:

**Scenario 1: Null Sessions Disabled**
```bash
rpcclient -U "" -N 192.168.1.100
Cannot connect to server. Error was NT_STATUS_ACCESS_DENIED
```
**Solution**: Try with valid credentials or check SMB security settings

**Scenario 2: SMB Service Down**
```bash
rpcclient -U "" -N 192.168.1.100
Cannot connect to server. Error was NT_STATUS_CONNECTION_REFUSED
```
**Solution**: Verify SMB service is running and port 445 is accessible

**Scenario 3: Firewall Blocking**
```bash
rpcclient -U "" -N 192.168.1.100
# Connection hangs or times out
```
**Solution**: Check firewall rules and network connectivity

### Performance Optimization:
```bash
# Faster enumeration with timeout
timeout 30 rpcclient -U "" -N <target> -c "enumdomusers; quit"

# Non-interactive batch processing
echo "enumdomusers" | timeout 15 rpcclient -U "" -N <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only test systems with explicit permission
- **Scope Compliance**: Remain within authorized testing boundaries
- **Data Sensitivity**: RPC enumeration reveals sensitive system information
- **Documentation**: Maintain detailed logs of all enumeration activities

### Operational Security:
- **Logging Impact**: RPC connections generate security logs on Windows
- **Detection Risk**: Null session attempts may trigger security alerts
- **Network Footprint**: RPC enumeration creates network traffic patterns
- **Timing**: Space out enumeration to avoid detection

### Detection Indicators:
RPC enumeration may be detected through:
- **Windows Event Logs**: Logon events (4624, 4625, 4672)
- **Security Logs**: Anonymous logon attempts
- **Network Monitoring**: SMB/RPC traffic patterns
- **IDS Signatures**: Null session connection attempts

### Defensive Countermeasures:
Organizations can protect against RPC enumeration by:
- **Disable Null Sessions**: Configure `RestrictAnonymous = 2`
- **SMB Hardening**: Disable SMBv1, enable signing
- **Network Segmentation**: Limit SMB access between subnets
- **Monitoring**: Deploy alerts for anonymous SMB connections
- **Account Policies**: Implement strong authentication requirements

### Risk Assessment:
| Activity | Detection Risk | Information Value | Impact |
|----------|----------------|-------------------|--------|
| Anonymous Connection Test | 🟡 Medium | 🟠 High | Security configuration check |
| User Enumeration | 🟠 High | 🔴 Critical | Identity disclosure |
| Group Enumeration | 🟠 High | 🔴 Critical | Permission structure |
| System Information | 🟡 Medium | 🟠 High | Fingerprinting data |

---

## 📖 References

### Official Documentation:
- [Samba rpcclient Manual](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) - Official tool documentation
- [Microsoft RPC Documentation](https://docs.microsoft.com/en-us/windows/win32/rpc/rpc-start-page) - Windows RPC reference
- [SMB Protocol Specifications](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/) - Technical protocol details

### Security Research:
- [Null Session Attacks](https://www.sans.org/reading-room/whitepapers/protocols/null-session-attacks-19) - SANS research paper
- [Windows RPC Vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=rpc) - CVE database
- [SMB Security Hardening](https://www.cisecurity.org/controls/secure-configuration-for-hardware-and-software/) - CIS controls

### Learning Resources:
- [Samba Administration Guide](https://www.samba.org/samba/docs/current/Samba-HOWTO-Collection/) - Comprehensive documentation
- [Windows RPC Internals](https://docs.microsoft.com/en-us/windows/win32/rpc/how-rpc-works) - Technical deep dive
- [SMB Enumeration Methodology](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) - Practical enumeration guide

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Test anonymous access
rpcclient -U "" -N <target>

# Batch enumeration
rpcclient -U "" -N <target> -c "enumdomusers; enumdomgroups; srvinfo; quit"

# Authenticated access
rpcclient -U <username> <target>
```

### Key RPC Commands:
```bash
# Inside rpcclient shell
enumdomusers          # List users
enumdomgroups         # List groups  
srvinfo              # System info
querydominfo         # Domain info
netshareenumall      # List shares
getdompwinfo         # Password policy
```

### Important RIDs:
- **500** - Administrator
- **501** - Guest
- **512** - Domain Admins
- **513** - Domain Users

### Follow-up Tools:
1. `smbclient` - Access discovered shares
2. `enum4linux` - Comprehensive enumeration
3. `crackmapexec` - Credential testing
4. `bloodhound` - Active Directory analysis

### Success Indicators:
- Interactive `rpcclient $>` prompt
- User/group enumeration successful  
- System information retrieved
- Share listing obtained

### Time Estimates:
- **Connection test**: 5-10 seconds
- **Basic enumeration**: 1-2 minutes
- **Comprehensive enumeration**: 5-10 minutes
- **Automated analysis**: 10-20 minutes

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting RPC enumeration activities.*
