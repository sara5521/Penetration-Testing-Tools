# 🧭 nmblookup - SMB NetBIOS Name Lookup

This section covers the nmblookup tool for NetBIOS name resolution and SMB host discovery using NetBIOS Name Service (NBNS).

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Tool Overview](#-tool-overview)
- [Prerequisites](#-prerequisites)
- [Basic NetBIOS Lookup](#-1-basic-netbios-lookup)
- [Advanced Queries](#-2-advanced-queries)
- [Output Analysis](#-3-output-analysis-and-interpretation)
- [Network Discovery](#-4-network-discovery-techniques)
- [Integration with Other Tools](#-5-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

The `nmblookup` tool from the Samba suite helps you:
- Discover NetBIOS hostnames of Windows/Samba machines
- Identify workgroup and domain names
- Map SMB-enabled hosts in the network
- Resolve NetBIOS names to IP addresses
- Gather information for SMB enumeration planning
- Verify NetBIOS service availability

---

## 🗂️ Tool Overview

| Attribute | Details |
|-----------|---------|
| **Tool Name** | nmblookup |
| **Package** | samba-common-bin / samba |
| **Protocol** | NetBIOS Name Service (NBNS) |
| **Transport** | UDP |
| **Default Port** | 137/udp |
| **Platform** | Linux/Unix |
| **Authentication** | Not required |

---

## 📋 Prerequisites

### Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install samba-common-bin

# CentOS/RHEL
sudo yum install samba-client

# Or full Samba suite
sudo apt install samba
```

### Network Requirements:
- NetBIOS service enabled on target (UDP 137)
- Network connectivity to target
- No firewall blocking NetBIOS traffic

### Quick Connectivity Test:
```bash
# Test NetBIOS port accessibility
nmap -sU -p 137 <target-ip>
```

---

## 🔧 nmblookup NetBIOS Enumeration

## 🖥️ 1. Basic NetBIOS Lookup

### Standard Status Query:
```bash
nmblookup -A <target-ip>
```

**📌 INE Lab Example:**
```bash
nmblookup -A demo.ine.local
```

**📸 Sample Output:**
```bash
Looking up status of 192.220.69.3
        SAMBA-RECON    <00> - <ACTIVE>
        SAMBA-RECON    <03> - <ACTIVE>
        SAMBA-RECON    <20> - <ACTIVE>
        **MSBROWSE**   <01> - <GROUP> <ACTIVE>
        RECONLABS      <00> - <GROUP> <ACTIVE>
        RECONLABS      <1e> - <GROUP> <ACTIVE>
        
        MAC Address = 00-00-00-00-00-00
```

### Name Resolution Query:
```bash
nmblookup <netbios-name>
```

**Example:**
```bash
nmblookup SAMBA-RECON
nmblookup WORKGROUP
```

**📸 Sample Output:**
```bash
192.220.69.3 SAMBA-RECON<00>
```

### Reverse Lookup:
```bash
nmblookup -T <target-ip>
```

---

## ⚙️ 2. Advanced Queries

### Command Line Options:

| Option | Description | Example |
|--------|-------------|---------|
| `-A` | Status query (most common) | `nmblookup -A 192.168.1.100` |
| `-T` | Translate IP to NetBIOS name | `nmblookup -T 192.168.1.100` |
| `-M` | Query master browser | `nmblookup -M WORKGROUP` |
| `-S` | Name query with status | `nmblookup -S HOSTNAME` |
| `-R` | Use broadcast for name resolution | `nmblookup -R HOSTNAME` |
| `-U` | Specify NetBIOS name server | `nmblookup -U 192.168.1.1 HOSTNAME` |
| `-B` | Specify broadcast address | `nmblookup -B 192.168.1.255 HOSTNAME` |
| `-v` | Verbose output | `nmblookup -v -A 192.168.1.100` |
| `-d` | Debug level (0-10) | `nmblookup -d 2 -A 192.168.1.100` |

### Master Browser Query:
```bash
nmblookup -M -- -
nmblookup -M WORKGROUP
```

**📸 Sample Output:**
```bash
192.168.1.10 __MSBROWSE__<01>
```

### Broadcast Queries:
```bash
# Query entire subnet
nmblookup -B 192.168.1.255 WORKGROUP

# Find all workgroup members
nmblookup -M WORKGROUP
```

### Verbose Status Query:
```bash
nmblookup -v -A <target-ip>
```

**📸 Enhanced Output:**
```bash
Looking up status of 192.220.69.3
received 6 names
        SAMBA-RECON    <00> - <ACTIVE> <WORKSTATION>
        SAMBA-RECON    <03> - <ACTIVE> <MESSENGER>
        SAMBA-RECON    <20> - <ACTIVE> <FILE_SERVER>
        **MSBROWSE**   <01> - <GROUP> <ACTIVE> <MASTER_BROWSER>
        RECONLABS      <00> - <GROUP> <ACTIVE> <WORKGROUP>
        RECONLABS      <1e> - <GROUP> <ACTIVE> <BROWSER_ELECTION>
        
        MAC Address = 02-42-C0-DC-45-03
```

---

## 🔍 3. Output Analysis and Interpretation

### NetBIOS Service Codes:

| Code | Service Type | Description | Security Implication |
|------|-------------|-------------|---------------------|
| `<00>` | Workstation | Computer name | 🟡 Host identification |
| `<01>` | Master Browser | Domain master browser | 🟠 Network mapping |
| `<03>` | Messenger | Messenger service | 🟢 Legacy service |
| `<06>` | RAS Server | Remote Access Service | 🟠 Remote access point |
| `<1B>` | Domain Master | Primary domain controller | 🔴 Critical target |
| `<1C>` | Domain Controller | Domain controller | 🔴 Critical target |
| `<1D>` | Master Browser | Master browser | 🟡 Network role |
| `<1E>` | Browser Election | Browser service | 🟢 Service discovery |
| `<1F>` | NetDDE | Network DDE service | 🟡 Data exchange |
| `<20>` | File Server | SMB file sharing | 🟠 File access point |
| `<21>` | RAS Client | RAS client service | 🟡 Remote access |

### Service Flags:

| Flag | Meaning | Implication |
|------|---------|------------|
| `<ACTIVE>` | Service is running | Available for enumeration |
| `<GROUP>` | Group name (domain/workgroup) | Network boundary |
| `<WORKSTATION>` | Workstation service | Standard computer |
| `<FILE_SERVER>` | File sharing enabled | SMB enumeration target |
| `<MASTER_BROWSER>` | Master browser role | Network discovery hub |

### Sample Analysis Scenarios:

**Scenario 1: Windows Workstation**
```bash
CLIENT-PC      <00> - <ACTIVE>
CLIENT-PC      <20> - <ACTIVE>  
CORP           <00> - <GROUP> <ACTIVE>
```
**Analysis:** Standard Windows workstation in CORP domain with file sharing

**Scenario 2: Domain Controller**
```bash
DC01           <00> - <ACTIVE>
DC01           <20> - <ACTIVE>
CORP           <1C> - <GROUP> <ACTIVE>
CORP           <1B> - <ACTIVE>
```
**Analysis:** Domain controller - high-value target for domain enumeration

**Scenario 3: Samba Server**
```bash
FILESERVER     <00> - <ACTIVE>
FILESERVER     <20> - <ACTIVE>
WORKGROUP      <00> - <GROUP> <ACTIVE>
__MSBROWSE__   <01> - <GROUP> <ACTIVE>
```
**Analysis:** Linux Samba server acting as file server and master browser

### Key Information Extraction:

**🏷️ Hostname Identification:**
- Primary computer name from `<00>` entry
- Used for SMB connections and credential attacks

**🏢 Domain/Workgroup Discovery:**
- Group entries indicate network membership
- Critical for Active Directory targeting

**🎯 Service Identification:**
- `<20>` indicates SMB file sharing availability
- `<1C>` identifies domain controllers

**🌐 Network Role Discovery:**
- Master browser services indicate network topology
- Critical for understanding network structure

---

## 🌐 4. Network Discovery Techniques

### Subnet NetBIOS Discovery:

**Method 1: IP Range Scanning**
```bash
#!/bin/bash
# NetBIOS discovery script
for ip in 192.168.1.{1..254}; do
    echo "Checking $ip..."
    timeout 3 nmblookup -A $ip 2>/dev/null | grep -E "Looking up|<00>|<20>" &
done
wait
```

**Method 2: Broadcast Discovery**
```bash
# Discover workgroup members
nmblookup -M -- -

# Find specific workgroup
nmblookup -M WORKGROUP
nmblookup -M CORP
```

**Method 3: Integration with Nmap**
```bash
# First discover live hosts with SMB
nmap -sn -PS139,445 192.168.1.0/24

# Then run nmblookup on discovered hosts
nmap -p 139,445 --open 192.168.1.0/24 | grep "Nmap scan report" | awk '{print $5}' | while read host; do
    echo "=== NetBIOS lookup for $host ==="
    nmblookup -A $host
done
```

### Automated Network Mapping:
```bash
#!/bin/bash
# Comprehensive NetBIOS network mapping

echo "=== NetBIOS Network Discovery ==="
echo "Date: $(date)"
echo

# Discover master browsers
echo "Master Browsers:"
nmblookup -M -- - 2>/dev/null | grep -v "name_query failed"

echo
echo "Detailed NetBIOS Information:"
echo "================================"

# Scan common IP ranges
for subnet in "192.168.1" "192.168.0" "10.0.0"; do
    echo "Scanning $subnet.0/24..."
    for i in {1..10}; do  # Limit for demo
        ip="$subnet.$i"
        result=$(timeout 2 nmblookup -A $ip 2>/dev/null)
        if [[ $? -eq 0 && ! -z "$result" ]]; then
            echo "=== $ip ==="
            echo "$result"
            echo
        fi
    done
done
```

---

## 🔗 5. Integration with Other Tools

### With Nmap Scripts:
```bash
# Nmap NetBIOS enumeration
nmap -sU -p 137 --script nbstat 192.168.1.0/24

# Compare with nmblookup results
nmap -p 445 --script smb-os-discovery 192.168.1.100
nmblookup -A 192.168.1.100
```

### With enum4linux:
```bash
# enum4linux uses nmblookup internally
enum4linux -n 192.168.1.100

# Manual verification
nmblookup -A 192.168.1.100
```

### With smbclient:
```bash
# Use discovered NetBIOS names
nmblookup -A 192.168.1.100  # Get hostname
smbclient -L //DISCOVERED-HOSTNAME -N
```

### With rpcclient:
```bash
# Connect using NetBIOS name
nmblookup -A 192.168.1.100  # Get hostname
rpcclient -U "" -N //DISCOVERED-HOSTNAME
```

### Data Processing Pipeline:
```bash
#!/bin/bash
# NetBIOS to SMB enumeration pipeline

TARGET_RANGE="192.168.1.0/24"
OUTPUT_DIR="netbios_discovery"
mkdir -p $OUTPUT_DIR

echo "=== NetBIOS Discovery Pipeline ==="

# Step 1: Discover hosts with NetBIOS
echo "Step 1: NetBIOS host discovery"
nmap -sU -p 137 --open $TARGET_RANGE | grep "Nmap scan report" | awk '{print $5}' > $OUTPUT_DIR/netbios_hosts.txt

# Step 2: Get NetBIOS information
echo "Step 2: NetBIOS enumeration"
while read host; do
    echo "Processing $host..."
    nmblookup -A $host > $OUTPUT_DIR/netbios_$host.txt 2>&1
    
    # Extract hostname for SMB enumeration
    hostname=$(grep "<00>" $OUTPUT_DIR/netbios_$host.txt | head -1 | awk '{print $1}')
    if [[ ! -z "$hostname" ]]; then
        echo "$host,$hostname" >> $OUTPUT_DIR/host_mapping.txt
    fi
done < $OUTPUT_DIR/netbios_hosts.txt

# Step 3: SMB enumeration on discovered hosts
echo "Step 3: SMB enumeration"
while IFS=',' read ip hostname; do
    echo "SMB enumeration of $hostname ($ip)"
    smbclient -L //$ip -N > $OUTPUT_DIR/smb_$ip.txt 2>&1 &
done < $OUTPUT_DIR/host_mapping.txt

wait
echo "Discovery complete. Results in $OUTPUT_DIR/"
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **No Response** | `name_query failed` | Check if NetBIOS is enabled on target |
| **Connection Refused** | Connection timeout | Verify UDP 137 is not blocked |
| **Permission Denied** | Access denied error | Normal for NetBIOS queries |
| **Tool Not Found** | `command not found` | Install samba-common-bin package |
| **Network Unreachable** | Network timeout | Check network connectivity |

### Debugging Commands:
```bash
# Test UDP 137 connectivity
nmap -sU -p 137 <target-ip>

# Verbose nmblookup
nmblookup -v -d 2 -A <target-ip>

# Check local NetBIOS configuration
cat /etc/samba/smb.conf | grep -i netbios

# Test broadcast connectivity
ping -b 192.168.1.255
```

### Common Error Messages:
```bash
# NetBIOS disabled or filtered
nmblookup: name_query failed to find name HOSTNAME<00>
# Solution: Target may have NetBIOS disabled

# Network connectivity issues
nmblookup: getaddrinfo: Name or service not known
# Solution: Check DNS resolution and network connectivity

# Timeout issues
nmblookup: query failed
# Solution: Increase timeout or check firewall rules
```

### Performance Optimization:
```bash
# Faster scanning with timeout
timeout 3 nmblookup -A <target-ip>

# Parallel processing
for ip in {1..10}; do
    timeout 2 nmblookup -A 192.168.1.$ip &
done
wait
```

### Alternative Tools:
```bash
# If nmblookup fails, try alternatives
nbtscan 192.168.1.0/24
nmap -sU -p 137 --script nbstat <target>
enum4linux -n <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only scan networks you own or have permission to test
- **Network Policy Compliance**: Respect organizational network usage policies
- **Data Privacy**: NetBIOS names may contain sensitive information
- **Responsible Disclosure**: Report vulnerabilities through appropriate channels

### Operational Security:
- **Logging Minimal**: NetBIOS queries generate minimal logs but are still recorded
- **Broadcast Traffic**: Broadcast queries may be more noticeable
- **Network Discovery**: Reveals network topology to defenders
- **Timing Considerations**: Rapid scanning may trigger monitoring alerts

### Detection Indicators:
NetBIOS enumeration may be detected through:
- **Network Logs**: UDP 137 traffic patterns
- **NetBIOS Logs**: Name query requests in Windows logs
- **IDS Signatures**: NetBIOS scanning patterns
- **Broadcast Monitoring**: Unusual broadcast traffic

### Defensive Countermeasures:
Organizations can protect against NetBIOS enumeration by:
- **Disable NetBIOS**: Turn off NetBIOS over TCP/IP
- **Firewall Rules**: Block UDP 137 at network boundaries
- **Network Segmentation**: Limit broadcast domains
- **Monitoring**: Deploy alerts for NetBIOS scanning activity

### Risk Assessment:
| Activity | Detection Risk | Information Value | Recommendation |
|----------|----------------|-------------------|----------------|
| Single Host Query | 🟢 Low | 🟡 Medium | Generally safe |
| Subnet Discovery | 🟡 Medium | 🟠 High | Use with caution |
| Broadcast Queries | 🟠 High | 🟠 High | Coordinate with team |
| Automated Scanning | 🔴 High | 🔴 High | Only with authorization |

---

## 📖 References

### Official Documentation:
- [Samba nmblookup Manual](https://www.samba.org/samba/docs/current/man-html/nmblookup.1.html) - Official tool documentation
- [NetBIOS Specification](https://tools.ietf.org/html/rfc1001) - RFC 1001/1002 NetBIOS standards
- [Microsoft NetBIOS](https://docs.microsoft.com/en-us/windows/win32/netbios/netbios-start-page) - Windows NetBIOS implementation

### Security Research:
- [NetBIOS Security Issues](https://www.sans.org/reading-room/whitepapers/protocols/netbios-name-service-security-issues-19) - SANS research
- [Windows NetBIOS Vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=netbios) - CVE database
- [Network Discovery Techniques](https://attack.mitre.org/techniques/T1018/) - MITRE ATT&CK framework

### Learning Resources:
- [Samba Administration Guide](https://www.samba.org/samba/docs/current/Samba-HOWTO-Collection/) - Comprehensive Samba documentation
- [NetBIOS Protocol Analysis](https://www.wireshark.org/docs/dfref/n/nbns.html) - Wireshark protocol reference
- [SMB Enumeration Methodology](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) - Practical enumeration guide

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Basic status query
nmblookup -A <target-ip>

# Name resolution
nmblookup <netbios-name>

# Master browser discovery
nmblookup -M -- -

# Verbose lookup
nmblookup -v -A <target-ip>
```

### Key NetBIOS Codes:
- `<00>` - Computer name (workstation)
- `<20>` - File server (SMB sharing)
- `<1C>` - Domain controller
- `<01>` - Master browser

### Follow-up Tools:
1. `smbclient -L //<hostname>` - SMB share enumeration
2. `enum4linux <target>` - Comprehensive enumeration
3. `rpcclient <target>` - RPC enumeration
4. `nmap --script smb-*` - SMB vulnerability scanning

### Information Priority:
1. **High Value**: Domain controllers (`<1C>`), file servers (`<20>`)
2. **Medium Value**: Workstations (`<00>`), master browsers (`<01>`)
3. **Low Value**: Legacy services (`<03>`, `<1F>`)

### Time Estimates:
- **Single host**: 5-10 seconds
- **Small network**: 1-3 minutes
- **Large subnet**: 10-30 minutes
- **Enterprise discovery**: 30+ minutes

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting network discovery activities.*
