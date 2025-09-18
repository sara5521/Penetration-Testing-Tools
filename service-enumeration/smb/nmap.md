# 📦 SMB/NetBIOS Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate SMB (Server Message Block) and NetBIOS services on Windows machines.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-smb-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [OS Discovery](#-1-discover-os-and-hostname)
- [Protocol Detection](#-2-detect-smb-protocol-version)
- [Security Mode](#-3-check-smb-security-mode)
- [Domain Enumeration](#-4-enumerate-smb-domains--workgroups)
- [User Enumeration](#-5-enumerate-users)
- [Group Enumeration](#-6-enumerate-user-groups)
- [Active Sessions](#-7-list-active-sessions)
- [Running Services](#-8-enumerate-running-services)
- [Share Enumeration](#-9-enumerate-and-browse-smb-shares)
- [Server Statistics](#-10-view-smb-server-stats)
- [NetBIOS Enumeration](#-11-netbios-service-enumeration)
- [Script Arguments](#-12-using---script-args-in-nmap)
- [Complete Assessment Workflow](#-complete-smb-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from SMB/NetBIOS services, such as:
- Operating system and hostname
- Shared folders (shares)
- Users and groups
- Domain or workgroup names
- Security configurations
- Server statistics and uptime
- Protocol versions
- Running services
- NetBIOS name information

---

## 🗂️ SMB Enumeration Scripts Summary

| # | Script                 | What it Does                             | Auth Required | Ports |
|---|------------------------|------------------------------------------|---------------|-------|
| 1 | smb-os-discovery       | Detect OS and hostname                   | ❌            | 139,445 |
| 2 | smb-protocols          | Show SMB versions supported              | ❌            | 139,445 |
| 3 | smb-security-mode      | Check SMB signing and auth level         | ❌            | 139,445 |
| 4 | smb-enum-domains       | List domain/workgroup info               | ✅            | 139,445 |
| 5 | smb-enum-users         | List user accounts                       | ✅            | 139,445 |
| 6 | smb-enum-groups        | List Windows groups                      | ✅            | 139,445 |
| 7 | smb-enum-sessions      | List active SMB sessions                 | ✅            | 139,445 |
| 8 | smb-enum-services      | List running services                    | ✅            | 139,445 |
| 9 | smb-enum-shares        | List shared folders                      | ⚠️*           | 139,445 |
| 10 | smb-ls                | List files inside shares                 | ⚠️*           | 139,445 |
| 11 | smb-server-stats      | Show stats like open files/sessions      | ✅            | 139,445 |
| 12 | nbstat                | NetBIOS name table enumeration           | ❌            | 137 |
| 13 | smb-vuln-*            | Vulnerability scanning                   | ❌            | 139,445 |

*⚠️ May work with anonymous access depending on target configuration*

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has SMB enabled** (port 445/tcp or 139/tcp open)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick Port Check:
```bash
# Check all SMB/NetBIOS related ports
nmap -p 135,137,139,445 <target-ip>

# Check specifically for SMB
nmap -p 445 <target-ip>
```

### Check if SMB is responsive:
```bash
nmap -p 445 --script smb-protocols <target-ip>
```

---

## 🔧 SMB/NetBIOS Enumeration Commands

## 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 139,445 --script smb-os-discovery <target-ip>
```
**INE Lab Example**
```bash
nmap --script smb-os-discovery -p 139,445 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: demo
|   NetBIOS computer name: SAMBA-RECON\x00
|   Domain name: ine.local
|   FQDN: demo.ine.local
|   System time: 2024-07-05T01:34:01+08:00
```
**Interpretation:**
- OS: Samba 4.3.11 running on Windows 6.1 (probably an emulated setup)
- Computer name: `demo`
- NetBIOS Name: `SAMBA-RECON`
- Domain: `ine.local`
- System Time: Useful for timestamp-based exploits or detecting timezone config

**Pro Tip:** This script rarely requires authentication and is great for initial reconnaissance.

---

## 🌐 2. Detect SMB Protocol Version
```bash
nmap -p 139,445 --script smb-protocols <target-ip>
```
**Purpose:**

This script shows which SMB version the server supports:
- SMBv1 → Old and not secure ⚠️
- SMBv2 / SMBv3 → Newer and more secure ✅

**Sample Output:**
```bash
Host script results:
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.02
|     2.10
|     3.00
|_    3.02
```

**Why is this useful?**
- If the server supports SMBv1, it's risky and may be vulnerable to attacks like EternalBlue
- Tells you whether modern, secure protocols (SMBv3) are supported
- Important for deciding what kind of attack or enumeration you can try

**Security Alert:** If SMBv1 is enabled, the target may be vulnerable to:
- MS17-010 (EternalBlue)
- MS08-067 (Netapi vulnerability)

---

## 🔐 3. Check SMB Security Mode
```bash
nmap -p 139,445 --script smb-security-mode <target-ip>
```
**Purpose:**

This script checks how secure the SMB server is by showing:
- If SMB signing is supported
- If SMB signing is required
- What type of authentication is used (user or share level)

**INE Lab Example:**
```bash
nmap -p445 --script smb-security-mode demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```
**Interpretation:**
- Account Used: Guest (anonymous)
- Auth Level: User-level (safer than share-level)
- Challenge/Response: Supported (adds some protection)
- Message Signing: Disabled (vulnerable to MITM attacks)

**Attack Vectors:** When message signing is disabled:
- SMB relay attacks possible
- Man-in-the-middle attacks feasible
- Consider using tools like `Responder` or `ntlmrelayx`
  
---

## 🏢 4. Enumerate SMB Domains / Workgroups
```bash
nmap -p 139,445 --script smb-enum-domains <target-ip>
```
**Purpose:**

This script tries to list the Windows domains or workgroups the SMB server belongs to.

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-domains --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-domains: 
|   WIN-OMCNBKR66MN
|     Groups: WinRMRemoteWMIUsers__
|     Users: Administrator, bob, Guest
|     Creation time: 2013-08-22T14:47:57
|     Passwords: min length: n/a; min age: n/a days; max age: 42 days; history: n/a passwords
|     Properties: Complexity requirements exist
|     Account lockout disabled
|   Builtin
|     Groups: Access Control Assistance Operators, Administrators, Backup Operators...
|     Users: n/a
|     Creation time: 2013-08-22T14:47:57
|_    Passwords: min length: n/a; min age: n/a days; max age: 42 days; history: n/a passwords
```

**Interpretation:**
- Domain name: `WIN-OMCNBKR66MN`
- Users: Administrator, bob, Guest
- Groups: WinRMRemoteWMIUsers__
- Password Policy:
  - Max age = 42 days
  - Complexity = Enabled
  - Lockout = Disabled ⚠️

**Attack Implications:**
- Account lockout disabled = Safe for brute force attacks
- Password complexity enabled = Need complex passwords
- 42-day max age = Look for accounts with old passwords

---

## 👥 5. Enumerate Users
```bash
nmap -p 139,445 --script smb-enum-users <target-ip>
```
**Purpose:**

This script tries to list user accounts on a Windows machine by querying the SMB service.

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-users --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-users: 
|   WIN-OMCNBKR66MN\Administrator (RID: 500)
|     Description: Built-in account for administering the computer/domain
|     Flags:       Normal user account, Password does not expire
|   WIN-OMCNBKR66MN\bob (RID: 1010)
|     Flags:       Normal user account, Password does not expire
|   WIN-OMCNBKR66MN\Guest (RID: 501)
|     Description: Built-in account for guest access to the computer/domain
|_    Flags:       Normal user account, Password not required, Password does not expire
```
**Interpretation:**
- Administrator is the default admin account — useful for brute-force attacks
- bob is likely a regular user account added by an admin — try brute-forcing it
- Guest has no password required — useful for anonymous logins
- RIDs (Relative IDs): 500 = Administrator, 501 = Guest, 1000+ = Custom users

**Next Steps:**
- Save usernames to a file for password attacks
- Check if Guest account has network access
- Look for service accounts (usually have "svc" prefix)

---

## 👤 6. Enumerate User Groups
```bash
nmap -p 139,445 --script smb-enum-groups <target-ip>
```
**Purpose:**

This script tries to list the Windows groups on the target — groups are like "roles" or "permission levels" for users.

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-groups --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-groups: 
|   Builtin\Administrators (RID: 544): Administrator, bob
|   Builtin\Users (RID: 545): bob
|   Builtin\Guests (RID: 546): Guest
|   Builtin\Remote Desktop Users (RID: 555): bob
|   Builtin\Backup Operators (RID: 551): <empty>
|   ...
|_  WIN-OMCNBKR66MN\WinRMRemoteWMIUsers__ (RID: 1000): <empty>
```
**Interpretation:**
- `Administrator` and `bob` are members of the Administrators group → full system privileges
- `bob` is also part of the Remote Desktop Users group → RDP access possible
- `Guest` is in the Guests group → usually has limited permissions

**High-Value Groups to Look For:**
- `Domain Admins` - Full domain control
- `Enterprise Admins` - Full forest control  
- `Backup Operators` - Can backup/restore files
- `Remote Desktop Users` - RDP access
- `WinRMRemoteWMIUsers` - PowerShell remoting

---

## 🔄 7. List Active Sessions
```bash
nmap -p 139,445 --script smb-enum-sessions <target-ip>
```
**Purpose:**

This script tries to list current SMB sessions showing who is currently connected to the SMB server.

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-sessions --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-sessions: 
|   Users logged in
|     WIN-OMCNBKR66MN\bob since 2025-09-09T09:43:25
|   Active SMB sessions
|_    ADMINISTRATOR is connected from \\10.10.45.4 for [just logged in, it's probably you], idle for [not idle]
```
**Interpretation:**
- bob is currently logged into the system
- His session has been active since 09:43:25
- The Administrator account is also logged in
- That session came from IP 10.10.45.4

**Operational Security:**
- If you see unexpected active sessions, the system might be monitored
- Multiple admin sessions could indicate other attackers

---

## ⚙️ 8. Enumerate Running Services
```bash
nmap -p 139,445 --script smb-enum-services <target-ip>
```
**Purpose:**

This script tries to list the Windows services that are running on the target through SMB.

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-services --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-services:
|   AmazonSSMAgent:
|     display_name: Amazon SSM Agent
|     state: SERVICE_RUNNING
|   Spooler:
|     display_name: Print Spooler
|     state: SERVICE_RUNNING
|   TrustedInstaller:
|     display_name: Windows Modules Installer
|     state: SERVICE_RUNNING
|   Ec2Config:
|     display_name: Ec2Config
|     state: SERVICE_RUNNING
```

**Services of Interest for Exploitation:**
- `Print Spooler` - PrintNightmare vulnerability
- `WebClient` - WebDAV attacks
- `MSSQL$*` - Database access
- Cloud agents (AWS SSM, EC2Config) - Indicates cloud environment

---

## 📂 9. Enumerate and Browse SMB Shares

### List shared folders:
```bash
nmap -p 139,445 --script smb-enum-shares <target-ip>
```
**Purpose:**

The `smb-enum-shares` script helps you discover shared folders on a target system.

**Multiple Authentication Methods:**
```bash
# Anonymous access test
nmap --script smb-enum-shares -p 139,445 <target>

# Guest access test
nmap --script smb-enum-shares --script-args smbusername=guest,smbpassword= -p 139,445 <target>

# Authenticated access
nmap --script smb-enum-shares --script-args smbusername=<user>,smbpassword=<pass> -p 139,445 <target>

# Check for anonymous access specifically
nmap --script smb-enum-shares --script-args smbusername="",smbpassword="" -p 139,445 <target>
```

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-shares --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-enum-shares:
|   account_used: administrator
|   \\10.6.18.229\ADMIN$       → Hidden admin share (C:\Windows) [READ/WRITE]
|   \\10.6.18.229\C            → Root of C:\ drive [READ]
|   \\10.6.18.229\C$           → Hidden admin share (C:\) [READ/WRITE]
|   \\10.6.18.229\D$           → Hidden admin share (D:\) [READ/WRITE]
|   \\10.6.18.229\Documents    → Admin Documents folder [READ]
|   \\10.6.18.229\Downloads    → Admin Downloads folder [READ]
|   \\10.6.18.229\IPC$         → Inter-process communication share [READ/WRITE]
|_  \\10.6.18.229\print$       → Printer drivers share [READ/WRITE]
```

### List files inside the shares:
```bash
nmap -p 139,445 --script smb-ls <target-ip>
```
```bash
# List specific directories
nmap --script smb-ls --script-args 'smbusername=<user>,smbpassword=<pass>,share=C$,path=\' -p 139,445 <target>

# Search in system directories
nmap --script smb-ls --script-args 'smbusername=<user>,smbpassword=<pass>,share=C$,path=\windows\system32\' -p 139,445 <target>
```

**Files to Look For:**
- `.txt`, `.log`, `.bak` files (potential sensitive data)
- `.exe`, `.bat`, `.ps1` files (potential malware upload locations)
- User folders (Desktop, Documents, Downloads)
- Database files (`.mdb`, `.sqlite`, `.db`)

---

## 📊 10. View SMB Server Stats
```bash
nmap -p 139,445 --script smb-server-stats <target-ip>
```
**Purpose:**

This script gets real-time stats from the SMB server.

**INE Lab Example:**
```bash
nmap -p445 --script smb-server-stats --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smb-server-stats: 
|   Server statistics collected since 2025-09-09T13:33:45 (4m19s):
|     2250 bytes (8.69 b/s) sent, 2141 bytes (8.27 b/s) received
|_    0 failed logins, 0 permission errors, 0 system errors, 0 print jobs, 2 files opened
```

**Red Flags to Watch For:**
- High number of failed logins → Possible brute force attack
- Many permission errors → Unauthorized access attempts
- Excessive open files → Potential data exfiltration

---

## 🔍 11. NetBIOS Service Enumeration

### NetBIOS Name Service (Port 137)
```bash
# NetBIOS name table enumeration
nmap --script nbstat -p 137 <target>

# NetBIOS name resolution
nmap --script nbstat --script-args nbstat.target=<target> -p 137 <target>

# Broadcast NetBIOS enumeration
nmap --script nbstat -p 137 <network_range>
```

**Purpose:**
NetBIOS enumeration reveals:
- Computer names and workgroup information
- Active services (Workstation, Server, Messenger)
- MAC addresses
- Network adapter information

**Sample Output:**
```bash
Host script results:
| nbstat: NetBIOS name: WIN-SERVER, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:xx:xx:xx (VMware)
| Names:
|   WIN-SERVER<00>        Flags: <unique><active>
|   WIN-SERVER<20>        Flags: <unique><active>
|   WORKGROUP<1e>         Flags: <group><active>
|   WORKGROUP<00>         Flags: <group><active>
|_  \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
```

### NetBIOS Session Service (Port 139)
```bash
# NetBIOS session enumeration
nmap --script smb-enum-sessions -p 139 <target>

# NetBIOS share enumeration via port 139
nmap --script smb-enum-shares -p 139 <target>

# Legacy NetBIOS testing
nmap --script smb-os-discovery -p 139 <target>
```

---

## 🧾 12. Using `--script-args` in Nmap
```bash
nmap -p 139,445 --script <script-name> --script-args <key>=<value>
```
**Purpose:**

`--script-args` lets you customize the behavior of NSE scripts by providing extra input.

### Examples with SMB:
**Login with username and password:**
```bash
nmap -p 139,445 --script smb-enum-users --script-args smbuser=admin,smbpass=123456 <target-ip>
```

**Specify domain:**
```bash
nmap -p 139,445 --script smb-enum-shares --script-args smbuser=user1,smbpass=pass123,smbdomain=WORKGROUP <target-ip>
```

**Try anonymous access:**
```bash
nmap -p 139,445 --script smb-enum-shares --script-args smbuser=guest,smbpass= <target-ip>
```

### Common --script-args keys for SMB:
| Key          | Description                    | Example Values           |
|------------- |--------------------------------|--------------------------|
| `smbuser`    | Username for SMB login         | `administrator`, `guest` |
| `smbpass`    | Password for SMB login         | `password123`, ` ` (empty) |
| `smbdomain`  | Windows domain or workgroup    | `WORKGROUP`, `CORP.LOCAL` |
| `timeout`    | Script timeout (e.g. 10s)     | `10s`, `30s`, `1m`       |
| `smbnoguest` | Disable guest account fallback | `true`, `false`          |

---

## 🎯 Complete SMB Assessment Workflow

### Phase 1: Discovery and Basic Information
```bash
# Step 1: Port discovery
nmap -p 135,137,139,445 <target>

# Step 2: Service version detection
nmap -sV -p 135,137,139,445 <target>

# Step 3: Basic SMB information
nmap --script smb-os-discovery,smb-security-mode -p 139,445 <target>

# Step 4: NetBIOS enumeration
nmap --script nbstat -p 137 <target>
```

### Phase 2: Share and Resource Enumeration
```bash
# Step 5: Anonymous share enumeration
nmap --script smb-enum-shares -p 139,445 <target>

# Step 6: User enumeration (if possible)
nmap --script smb-enum-users -p 139,445 <target>

# Step 7: Protocol and capability detection
nmap --script smb-protocols -p 139,445 <target>
```

### Phase 3: Security Assessment
```bash
# Step 8: Vulnerability scanning
nmap --script smb-vuln-* -p 139,445 <target>

# Step 9: Authentication testing
nmap --script smb-brute -p 139,445 <target>

# Step 10: Advanced enumeration (with credentials if obtained)
nmap --script smb-enum-shares,smb-ls --script-args smbusername=<user>,smbpassword=<pass> -p 139,445 <target>
```

### Critical SMB Vulnerabilities to Check
```bash
# EternalBlue (MS17-010) - Critical
nmap --script smb-vuln-ms17-010 -p 445 <target>

# SMBGhost (CVE-2020-0796) - Critical
nmap --script smb-vuln-cve2009-3103 -p 445 <target>

# MS08-067 NetAPI vulnerability
nmap --script smb-vuln-ms08-067 -p 445 <target>
```

---

## 🔁 All-in-One Scan

### Basic scan (no authentication)
```bash
nmap -p 135,137,139,445 --script "smb-os-discovery,smb-protocols,smb-security-mode,nbstat" <target-ip>
```

### Authenticated scan (with credentials)
```bash
nmap -p 135,137,139,445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats,nbstat" --script-args smbuser=administrator,smbpass=password123 <target-ip>
```

### Anonymous/Guest scan
```bash
nmap -p 135,137,139,445 --script "smb-enum-shares,smb-ls,nbstat" --script-args smbuser=guest,smbpass= <target-ip>
```

### Vulnerability Assessment
```bash
nmap -p 139,445 --script smb-vuln-* <target-ip>
```

### Comprehensive Assessment
```bash
# Complete Windows networking scan
nmap -sS -sV -p 135,137,139,445 <target>

# Full SMB enumeration with authentication
nmap --script smb-* --script-args smbuser=<user>,smbpass=<pass> -p 139,445 <target>

# Performance optimized scan
nmap -T4 --script smb-os-discovery,smb-enum-shares,smb-vuln-ms17-010 -p 139,445 <target>
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Permission Denied** | `NT_STATUS_ACCESS_DENIED` | Try with credentials using `--script-args` |
| **Connection Refused** | `Connection refused` | Check if SMB service is running on target |
| **Timeout Issues** | Scripts hang or timeout | Use `--script-args timeout=30s` |
| **Empty Results** | No shares/users found | Target might block anonymous access |
| **Authentication Failed** | `NT_STATUS_LOGON_FAILURE` | Verify username/password combination |

### Common Error Messages:
- `NT_STATUS_ACCESS_DENIED` → Need valid credentials
- `NT_STATUS_LOGON_FAILURE` → Wrong username/password
- `NT_STATUS_ACCOUNT_DISABLED` → Account is disabled
- `NT_STATUS_PASSWORD_EXPIRED` → Password needs reset
- `NT_STATUS_NETWORK_UNREACHABLE` → Network/firewall issue

### Performance Optimization:
```bash
# Faster scanning with timing templates
nmap -p 139,445 --script smb-enum-shares -T4 <target-ip>

# Parallel script execution
nmap -p 139,445 --script "smb-enum-shares,smb-enum-users" --script-args timeout=10s <target-ip>

# Comprehensive but optimized
nmap -T3 --script smb-* --script-args smb.timeout=5 -p 139,445 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **Follow responsible disclosure** if vulnerabilities are found
- **Respect system availability** - don't crash production systems

### Operational Security (OpSec):
- **Logging Awareness**: These commands generate logs on target systems
- **Network Detection**: SMB enumeration can trigger IDS/IPS alerts
- **Timing**: Consider using slower scan rates (`-T1`, `-T2`) to avoid detection

### Detection Indicators:
Target systems may detect SMB enumeration through:
- **Event Logs**: Windows Security logs (Event ID 4624, 4625, 4648)
- **Network Monitoring**: Unusual SMB traffic patterns
- **Failed Login Attempts**: Multiple authentication failures
- **Process Monitoring**: SMB-related process spawning

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| SMBv1 Enabled | 🔴 Critical | Disable immediately |
| Guest Account Active | 🟡 Medium | Disable or restrict |
| No Message Signing | 🟠 High | Enable signing requirement |
| Writable Admin Shares | 🔴 Critical | Review share permissions |
| Weak Passwords | 🟠 High | Implement password policy |

---

## 📖 Real Lab Examples

### Example 1: Windows Domain Controller Assessment
```bash
# Target: Domain Controller (192.168.1.10)

# Basic discovery
nmap -sS -sV -p 135,137,139,445 192.168.1.10

# OS and domain information
nmap --script smb-os-discovery -p 139,445 192.168.1.10

# NetBIOS enumeration
nmap --script nbstat -p 137 192.168.1.10

# Share enumeration
nmap --script smb-enum-shares -p 139,445 192.168.1.10

# User enumeration
nmap --script smb-enum-users -p 139,445 192.168.1.10

# Vulnerability assessment
nmap --script smb-vuln-* -p 139,445 192.168.1.10
```

### Example 2: Vulnerable Windows System Assessment
```bash
# Target: Legacy Windows system with potential vulnerabilities

# EternalBlue detection
nmap --script smb-vuln-ms17-010 -p 445 192.168.1.50

# Multiple vulnerability assessment
nmap --script smb-vuln-* -p 139,445 192.168.1.50

# Share enumeration with anonymous access
nmap --script smb-enum-shares --script-args smbusername=guest,smbpassword= -p 139,445 192.168.1.50
```

---

## 🔗 Integration with Other Tools

After Nmap SMB enumeration, follow up with:
- **smbclient:** Manual SMB share access and file operations
- **enum4linux:** Comprehensive SMB/NetBIOS enumeration
- **rpcclient:** RPC service interaction
- **crackmapexec:** SMB credential spraying and lateral movement
- **smbmap:** SMB share enumeration and file searching
- **Metasploit:** SMB exploitation modules

---

## 🎯 eJPT Focus Points

### Essential SMB Commands for Certification:
```bash
# Basic SMB discovery
nmap --script smb-os-discovery -p 139,445 <target>

# Share enumeration
nmap --script smb-enum-shares -p 139,445 <target>

# Vulnerability assessment
nmap --script smb-vuln-ms17-010 -p 445 <target>

# User enumeration
nmap --script smb-enum-users -p 139,445 <target>

# NetBIOS enumeration
nmap --script nbstat -p 137 <target>
```

### Key SMB Attack Vectors:
1. **Anonymous/Null Sessions:** Test with empty credentials
2. **Share Access:** Look for writable shares or sensitive data
3. **EternalBlue:** Always check MS17-010 on Windows systems
4. **User Enumeration:** Extract usernames for password attacks
5. **Domain Information:** Gather domain/workgroup details
6. **NetBIOS Information:** Collect computer names and services

---

## 📝 Output and Documentation

### Saving SMB Enumeration Results
```bash
# Comprehensive SMB assessment with output
nmap --script smb-* -oA smb_enumeration -p 135,137,139,445 <target>

# Specific vulnerability scan with output
nmap --script smb-vuln-* -oN smb_vulnerabilities.txt -p 445 <target>

# Share enumeration with XML output
nmap --script smb-enum-shares -oX smb_shares.xml -p 139,445 <target>

# NetBIOS enumeration with output
nmap --script nbstat -oN netbios_info.txt -p 137 <target>
```

---

## 🔍 Common SMB Ports and Services

### Standard SMB/NetBIOS Ports
```bash
# Complete Windows networking scan
nmap -sS -sV -p 135,137,139,445 <target>

# Port 137: NetBIOS Name Service
nmap --script nbstat -p 137 <target>

# Port 139: NetBIOS Session Service  
nmap --script smb-* -p 139 <target>

# Port 445: SMB over TCP (modern)
nmap --script smb-* -p 445 <target>

# Port 135: RPC Endpoint Mapper
nmap -sV -p 135 <target>
```

### Extended Windows Service Enumeration
```bash
# Full Windows service scan
nmap -sS -sV -p 135,137,139,445,593,3389,5985,5986 <target>

# RPC services
nmap --script rpc-grind -p 135 <target>

# WinRM enumeration
nmap --script winrm-* -p 5985,5986 <target>
```

---

## 🛡 Advanced SMB Security Assessment

### Authentication Testing Methods
```bash
# SMB brute force authentication
nmap --script smb-brute -p 139,445 <target>

# Test with specific credentials
nmap --script smb-enum-shares --script-args smbusername=<user>,smbpassword=<pass> -p 139,445 <target>

# Test with credential list
nmap --script smb-brute --script-args userdb=users.txt,passdb=passwords.txt -p 139,445 <target>

# Null session testing
nmap --script smb-enum-shares --script-args smbusername="",smbpassword="" -p 139,445 <target>

# Domain authentication
nmap --script smb-enum-shares --script-args smbdomain=DOMAIN,smbusername=user,smbpassword=pass -p 139,445 <target>

# Local authentication
nmap --script smb-enum-shares --script-args smbusername=.\user,smbpassword=pass -p 139,445 <target>
```

### Advanced Registry and Service Enumeration
```bash
# Registry key enumeration (requires authentication)
nmap --script smb-enum-services --script-args smbusername=<user>,smbpassword=<pass> -p 139,445 <target>

# SMB print spooler enumeration
nmap --script smb-print-text -p 139,445 <target>

# SMB statistics gathering
nmap --script smb-stat -p 139,445 <target>

# Process enumeration (if privileged access available)
nmap --script smb-enum-processes -p 139,445 <target>
```

---

## 🚨 SMB Vulnerability Assessment

### Critical SMB Vulnerabilities
```bash
# Comprehensive SMB vulnerability scan
nmap --script smb-vuln-* -p 139,445 <target>

# EternalBlue (MS17-010) - Critical
nmap --script smb-vuln-ms17-010 -p 445 <target>

# SMBGhost (CVE-2020-0796) - Critical  
nmap --script smb-vuln-cve2009-3103 -p 445 <target>

# MS08-067 NetAPI vulnerability
nmap --script smb-vuln-ms08-067 -p 445 <target>

# SMB signing detection
nmap --script smb-vuln-regsvc-dos -p 139,445 <target>

# Double Pulsar backdoor detection
nmap --script smb-vuln-ms17-010 --script-args vulns.showall -p 445 <target>
```

### Authentication and Access Issues
```bash
# Check for world-writable shares
nmap --script smb-enum-shares --script-args smbusername=guest,smbpassword= -p 139,445 <target>

# Test anonymous access
nmap --script smb-enum-users,smb-enum-shares -p 139,445 <target>

# Verify SMB signing requirements
nmap --script smb-security-mode -p 445 <target>
```

---

## 📊 Lab Practice Scenarios

### Scenario 1: Corporate Network Assessment
```bash
# Network: 192.168.1.0/24
# Target: Multiple Windows systems

# Phase 1: Network discovery
nmap -sn 192.168.1.0/24

# Phase 2: SMB service discovery
nmap -p 445 --open 192.168.1.0/24

# Phase 3: Detailed enumeration per target
for ip in $(nmap -p 445 --open 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+'); do
    echo "Scanning $ip"
    nmap --script smb-os-discovery,smb-enum-shares,smb-vuln-ms17-010 -p 445 $ip
done
```

### Scenario 2: Domain Controller Analysis
```bash
# Target: 192.168.1.10 (Suspected Domain Controller)

# Step 1: Port scan
nmap -sS -sV -p 53,88,135,139,389,445,464,636,3268,3269 192.168.1.10

# Step 2: SMB enumeration
nmap --script smb-os-discovery,smb-security-mode -p 445 192.168.1.10

# Step 3: Domain enumeration
nmap --script smb-enum-domains,smb-enum-users -p 445 192.168.1.10

# Step 4: Vulnerability check
nmap --script smb-vuln-* -p 445 192.168.1.10
```

### Scenario 3: Legacy System Assessment
```bash
# Target: 192.168.1.50 (Windows XP/2003 suspected)

# Check for legacy vulnerabilities
nmap --script smb-vuln-ms08-067,smb-vuln-ms17-010 -p 445 192.168.1.50

# Test anonymous access
nmap --script smb-enum-shares --script-args smbusername=,smbpassword= -p 139,445 192.168.1.50

# NetBIOS enumeration
nmap --script nbstat -p 137 192.168.1.50

# Full legacy assessment
nmap --script smb-* -p 139,445 192.168.1.50
```

---

## 🎯 Quick Reference Commands

### Fast Assessment Commands:
```bash
# Quick recon (no auth required)
nmap -p 445 --script smb-os-discovery,smb-security-mode <target>

# Guest access test
nmap -p 445 --script smb-enum-shares --script-args smbuser=guest,smbpass= <target>

# Full authenticated scan
nmap -p 445 --script "smb-enum-*" --script-args smbuser=admin,smbpass=pass <target>

# Critical vulnerability check
nmap -p 445 --script smb-vuln-ms17-010,smb-vuln-ms08-067 <target>

# NetBIOS quick check
nmap -p 137 --script nbstat <target>
```

### Time-Optimized Scans:
```bash
# Fast scan (2-3 minutes)
nmap -T4 --script smb-os-discovery,smb-enum-shares -p 445 <target>

# Medium scan (5-10 minutes)  
nmap -T3 --script smb-enum-*,smb-vuln-ms17-010 -p 139,445 <target>

# Comprehensive scan (15+ minutes)
nmap -T2 --script smb-* -p 135,137,139,445 <target>
```

---

## 📋 Command Cheat Sheet

### Essential Commands:
| Purpose | Command |
|---------|---------|
| OS Discovery | `nmap --script smb-os-discovery -p 445 <target>` |
| Share Enumeration | `nmap --script smb-enum-shares -p 445 <target>` |
| User Enumeration | `nmap --script smb-enum-users -p 445 <target>` |
| Vulnerability Scan | `nmap --script smb-vuln-* -p 445 <target>` |
| NetBIOS Info | `nmap --script nbstat -p 137 <target>` |
| Security Mode | `nmap --script smb-security-mode -p 445 <target>` |

### Authentication Variants:
| Access Type | Script Args |
|-------------|-------------|
| Anonymous | `--script-args smbusername="",smbpassword=""` |
| Guest | `--script-args smbusername=guest,smbpassword=` |
| Authenticated | `--script-args smbusername=user,smbpassword=pass` |
| Domain | `--script-args smbdomain=DOMAIN,smbusername=user,smbpassword=pass` |

---

## 🏆 Best Practices Summary

### Pre-Assessment:
1. Always verify target scope and authorization
2. Start with port discovery: `nmap -p 135,137,139,445 <target>`
3. Check service versions: `nmap -sV -p 445 <target>`

### Assessment Order:
1. **Basic Info**: OS discovery, security mode, protocols
2. **Anonymous Testing**: Guest access, null sessions
3. **Vulnerability Scanning**: EternalBlue, MS08-067, etc.
4. **Authenticated Enumeration**: Users, groups, shares, services
5. **Deep Analysis**: File listings, statistics, sessions

### Documentation:
1. Save all output: Use `-oA filename` for multiple formats
2. Screenshot interesting findings
3. Note any credentials discovered
4. Document attack paths and recommendations

### OpSec Considerations:
1. Use appropriate timing: `-T1` or `-T2` for stealth
2. Monitor for detection: Check logs if possible
3. Limit impact: Avoid DoS scripts in production
4. Clean up: Remove any uploaded files or changes

---

## 📚 Additional Resources

### SMB Protocol References:
- [Microsoft SMB Protocol Documentation](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview)
- [Samba Project Documentation](https://www.samba.org/samba/docs/)

### Security Research:
- [MS17-010 EternalBlue Analysis](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144)
- [SMB Security Best Practices](https://www.sans.org/white-papers/)

### Related Tools:
- [enum4linux](https://github.com/CiscoCXSecurity/enum4linux) - Comprehensive enumeration
- [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) - SMB client
- [crackmapexec](https://github.com/Porchetta-Industries/CrackMapExec) - Network protocol attacks

---

**Last Updated**: September 2025  
**Version**: 3.0  
**Scope**: Complete SMB/NetBIOS Enumeration Guide
