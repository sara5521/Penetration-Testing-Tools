# 📦 Nmap - SMB Enumeration Scripts

This section includes useful Nmap scripts to enumerate SMB (Server Message Block) services on Windows machines.

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
- [Script Arguments](#-11-using---script-args-in-nmap)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from the SMB service, such as:
- Operating system and hostname
- Shared folders (shares)
- Users and groups
- Domain or workgroup names
- Security configurations
- Server statistics and uptime
- Protocol versions
- Running services

---

## 🗂️ SMB Enumeration Scripts Summary

| # | Script                 | What it Does                             | Auth Required |
|---|------------------------|------------------------------------------|---------------|
| 1 | smb-os-discovery       | Detect OS and hostname                   | ❌            |
| 2 | smb-protocols          | Show SMB versions supported              | ❌            |
| 3 | smb-security-mode      | Check SMB signing and auth level         | ❌            |
| 4 | smb-enum-domains       | List domain/workgroup info               | ✅            |
| 5 | smb-enum-users         | List user accounts                       | ✅            |
| 6 | smb-enum-groups        | List Windows groups                      | ✅            |
| 7 | smb-enum-sessions      | List active SMB sessions                 | ✅            |
| 8 | smb-enum-services      | List running services                    | ✅            |
| 9 | smb-enum-shares        | List shared folders                      | ⚠️*           |
| 10 | smb-ls                | List files inside shares                 | ⚠️*           |
| 11 | smb-server-stats      | Show stats like open files/sessions      | ✅            |
| 12 | --script-args         | Customize script input (username, etc.)  | N/A           |

*⚠️ May work with anonymous access depending on target configuration*

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has SMB enabled** (port 445/tcp open)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick Port Check:
```bash
nmap -p 445 <target-ip>
```

### Check if SMB is responsive:
```bash
nmap -p 445 --script smb-protocols <target-ip>
```

---

## 🔧 Common SMB Nmap Scripts

## 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```
**INE Lab Example**
```bash
nmap --script smb-os-discovery -p 445 demo.ine.local
```
**📸 Sample Output:**
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
- 🖥️ OS: Samba 4.3.11 running on Windows 6.1 (probably an emulated setup)
- 💻 Computer name: `demo`
- 🧾 NetBIOS Name: `SAMBA-RECON`
- 🏢 Domain: `ine.local`
- ⏰ System Time: Useful for timestamp-based exploits or detecting timezone config

**💡 Pro Tip:** This script rarely requires authentication and is great for initial reconnaissance.

---

## 🌐 2. Detect SMB Protocol Version
```bash
nmap -p 445 --script smb-protocols <target-ip>
```
**📌 Purpose:**

This script shows which SMB version the server supports:
- SMBv1 → Old and not secure ⚠️
- SMBv2 / SMBv3 → Newer and more secure ✅

**📸 Sample Output:**
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

**🧠 Why is this useful?**
- If the server supports SMBv1, it's risky and may be vulnerable to attacks like EternalBlue
- Tells you whether modern, secure protocols (SMBv3) are supported
- Important for deciding what kind of attack or enumeration you can try

**🚨 Security Alert:** If SMBv1 is enabled, the target may be vulnerable to:
- MS17-010 (EternalBlue)
- MS08-067 (Netapi vulnerability)

---

## 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```
**📌 Purpose:**

This script checks how secure the SMB server is by showing:
- If SMB signing is supported
- If SMB signing is required
- What type of authentication is used (user or share level)

**🧠 Why is this useful?**
- If signing is not required, the server might be vulnerable to spoofing or MiTM (man-in-the-middle) attacks
- Helps you understand how the target is protected

**INE Lab Example:**
```bash
nmap -p445 --script smb-security-mode demo.ine.local
```
**📸 Sample Output:**
```bash
Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```
**Interpretation:**
- 🔑 Account Used: Guest (anonymous)
- 🛡️ Auth Level: User-level (safer than share-level)
- 🔁 Challenge/Response: Supported (adds some protection)
- ⚠️ Message Signing: Disabled (vulnerable to MITM attacks)

**🎯 Attack Vectors:** When message signing is disabled:
- SMB relay attacks possible
- Man-in-the-middle attacks feasible
- Consider using tools like `Responder` or `ntlmrelayx`
  
---

## 🏢 4. Enumerate SMB Domains / Workgroups
```bash
nmap -p 445 --script smb-enum-domains <target-ip>
```
**📌 Purpose:**

This script tries to list the Windows domains or workgroups the SMB server belongs to.

Think of a domain as the "name of the Windows network" (like `CORP`, `DEMO.LOCAL`, or `WORKGROUP`).

**🧠 Why is this useful?**
- Helps you know what domain or workgroup the machine belongs to
- Useful for:
  - Active Directory attacks
  - Targeting specific domain users
  - Mapping the Windows network
- Sometimes reveals the domain SID (Security Identifier)

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-domains --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**📸 Sample Output:**
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
|     Groups: Access Control Assistance Operators, Administrators, Backup Operators, Certificate Service DCOM Access, Cryptographic Operators, Distributed COM Users, Event Log Readers, Guests, Hyper-V Administrators, IIS_IUSRS, Network Configuration Operators, Performance Log Users, Performance Monitor Users, Power Users, Print Operators, RDS Endpoint Servers, RDS Management Servers, RDS Remote Access Servers, Remote Desktop Users, Remote Management Users, Replicator, Users
|     Users: n/a
|     Creation time: 2013-08-22T14:47:57
|_    Passwords: min length: n/a; min age: n/a days; max age: 42 days; history: n/a passwords
```

**Interpretation:**
- 💻 Domain name: `WIN-OMCNBKR66MN`
- 👥 Users: Administrator, bob, Guest
- 👮‍♂️ Groups: WinRMRemoteWMIUsers__
- ⏱️ Password Policy:
  - Max age = 42 days
  - Complexity = Enabled
  - Lockout = Disabled ⚠️
- 🧱 Builtin Groups: Show default permission roles like Admins, Guests, RDP users, etc.
- 🕓 Domain created: 2013-08-22

**🎯 Attack Implications:**
- Account lockout disabled = Safe for brute force attacks
- Password complexity enabled = Need complex passwords
- 42-day max age = Look for accounts with old passwords

---

## 👥 5. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```
**📌 Purpose:**

This script tries to list user accounts on a Windows machine by querying the SMB service.

**🧠 Why is this useful?**
- It gives you a list of usernames that exist on the system
- You can use these names for:
  - Brute-force attacks (e.g. with Hydra)
  - Password spraying
  - Understanding roles and access levels
- Sometimes even works with anonymous access

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-users --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
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
- 🔐 Administrator is the default admin account — useful for brute-force attacks
- 👨‍💻 bob is likely a regular user account added by an admin — try brute-forcing it
- 🧳 Guest has:
  - No password required ❗
  - Useful for anonymous logins and privilege escalation
- 🔢 RIDs (Relative IDs):
  - 500 = Administrator
  - 501 = Guest
  - 1000+ = Custom user accounts

**💡 Next Steps:**
- Save usernames to a file for password attacks
- Check if Guest account has network access
- Look for service accounts (usually have "svc" prefix)

---

## 👤 6. Enumerate User Groups
```bash
nmap -p 445 --script smb-enum-groups <target-ip>
```
**📌 Purpose:**

This script tries to list the Windows groups on the target — groups are like "roles" or "permission levels" for users.

**🧠 Why is this useful?**
- Helps you understand user roles and privileges
- Can reveal if there's a "Remote Desktop Users" group (target for RDP access)
- Useful for:
  - Privilege escalation
  - Brute-force targeting
  - Mapping group memberships in Active Directory

#### INE Lab Example:
```bash
nmap -p445 --script smb-enum-groups --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
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
- `Administrator` and `bob` are members of the Administrators group → they have full system privileges
- `bob` is also part of the Remote Desktop Users group → this means he may log in via RDP (Remote Desktop Protocol), which can be a target for brute-force attacks
- `Guest` is in the Guests group → usually has limited permissions and is less useful for privilege escalation
- Other groups like `Backup Operators`, `IIS_IUSRS`, etc., are empty → you can ignore them unless you find users assigned to them later

💡 **TIP**: This group membership information is useful for:
- **Privilege escalation** — targeting users with high privileges
- **Lateral movement** — if RDP or other remote services are enabled
- **Brute-force attacks** — focus on privileged accounts like `bob`

**🎯 High-Value Groups to Look For:**
- `Domain Admins` - Full domain control
- `Enterprise Admins` - Full forest control  
- `Backup Operators` - Can backup/restore files
- `Remote Desktop Users` - RDP access
- `WinRMRemoteWMIUsers` - PowerShell remoting

---

## 🔄 7. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```
**📌 Purpose:**

This script tries to list current SMB sessions — in other words, it shows:
- Who is currently connected to the SMB server
- What users or machines are active
- How many open sessions exist

**🧠 Why is this useful?**
- Shows real-time activity on the target
- Helps identify:
  - Logged-in users
  - IPs of other attackers or admins
  - If the system is actively being used

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-sessions --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
```bash
Host script results:
| smb-enum-sessions: 
|   Users logged in
|     WIN-OMCNBKR66MN\bob since 2025-09-09T09:43:25
|   Active SMB sessions
|_    ADMINISTRATOR is connected from \\10.10.45.4 for [just logged in, it's probably you], idle for [not idle]
```
**Interpretation:**
- 👤 bob is currently logged into the system
- ⏱️ His session has been active since 09:43:25
- 🧑‍💼 The Administrator account is also logged in — likely you
- 🖥️ That session came from IP 10.10.45.4
- 🔄 It is currently active, not idle

**⚠️ Operational Security:**
- If you see unexpected active sessions, the system might be monitored
- Multiple admin sessions could indicate other attackers
- Consider timing your attacks when system is less active

---

## ⚙️ 8. Enumerate Running Services
```bash
nmap -p 445 --script smb-enum-services <target-ip>
```
**📌 Purpose:**

This script tries to list the Windows services that are running on the target through SMB.

Think of services as background programs like:
- Remote Desktop
- Windows Update
- Print Spooler
- Antivirus
- Cloud Agent (e.g. SSM, EC2Config)

**🧠 Why is this useful?**
- Shows which services are:
  - Running
  - Paused
  - Accepting commands (e.g., stop, restart)
- Helps identify:
  - Possible privilege escalation paths
  - Cloud provider usage (e.g., AWS, Azure)
  - Entry points for exploitation (e.g., Print Spooler)

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-services --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**📸 Sample Output:**
```bash
Host script results:
| smb-enum-services:
|   AmazonSSMAgent:
|     display_name: Amazon SSM Agent
|     state:
|       SERVICE_RUNNING
|     ...
|   Spooler:
|     display_name: Print Spooler
|     state:
|       SERVICE_RUNNING
|   TrustedInstaller:
|     display_name: Windows Modules Installer
|     state:
|       SERVICE_RUNNING
|   DiagTrack:
|     display_name: Diagnostics Tracking Service
|     state:
|       SERVICE_RUNNING
|   Ec2Config:
|     display_name: Ec2Config
|     state:
|       SERVICE_RUNNING
|   sppsvc:
|     display_name: Software Protection
|     state:
|       SERVICE_RUNNING
|   vds:
|     display_name: Virtual Disk
|     state:
|_      SERVICE_RUNNING
```

**Interpretation:**
- ✅ The script successfully lists Windows Services that are actively running or paused
- Notable running services:
  - AmazonSSMAgent → Indicates this might be a cloud-hosted VM (e.g., AWS EC2)
  - Ec2Config → Confirms it's an AWS instance
  - Print Spooler (Spooler) → Frequently targeted for privilege escalation
  - TrustedInstaller, Software Protection (sppsvc) → Core Windows services
- SERVICE_RUNNING shows the service is active. You may be able to abuse some of them

**🎯 Services of Interest for Exploitation:**
- `Print Spooler` - PrintNightmare vulnerability
- `WebClient` - WebDAV attacks
- `MSSQL$*` - Database access
- `*Antivirus*` - EDR/AV detection

---

## 📂 9. Enumerate and Browse SMB Shares

### List shared folders:
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```
**📌 Purpose:**

The `smb-enum-shares` script helps you discover shared folders on a target system, showing which ones are accessible and whether they are hidden or writable.

**🧠 Why Is This Useful?**
- ✅ Discover accessible and hidden shares
- 🔍 Reveal sensitive or misconfigured folders
- ✍️ Identify writable locations to upload payloads
- 🔐 Great for post-exploitation, data gathering, or privilege escalation

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-shares --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**📸 Sample Output:**
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
**Interpretation:**

| Share Name        | Description                                        | Access       | Risk Level |
|-------------------|----------------------------------------------------|--------------|------------|
| `ADMIN$`          | Hidden remote admin share → points to `C:\Windows` | Read/Write   | 🔴 High    |
| `C$` / `D$`       | Hidden root shares for drives                      | Read/Write   | 🔴 High    |
| `C:`              | Normal (non-hidden) root share                     | Read         | 🟡 Medium  |
| `Documents`       | Admin's `Documents` folder                         | Read         | 🟡 Medium  |
| `Downloads`       | Admin's `Downloads` folder                         | Read         | 🟡 Medium  |
| `IPC$`            | Used for remote management and communication       | Read/Write   | 🟢 Low     |
| `print$`          | Shared printer driver folder                       | Read/Write   | 🟡 Medium  |

### List files inside the shares:
```bash
nmap -p 445 --script smb-ls <target-ip>
```
**📌 Purpose:**

The `smb-ls` script lists contents (files and folders) within each SMB share detected on the target. This is especially useful after identifying shares with `smb-enum-shares`.

**🧠 Why is this useful?**
- 🔍 Explore directories for sensitive files (`flag.txt`, `backup.zip`, etc.)
- 🪪 Identify writable shares for potential upload or exploitation
- 📦 Discover files that indicate software, tools, or misconfigurations

**INE Lab Example:**
```bash
nmap -p445 --script smb-enum-shares,smb-ls --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**📁 Key Findings Summary**

🔸 **ADMIN$ Share** (`C:\Windows`)
- Contains Windows system files
- Look for: `system32\`, `temp\`, `logs\`

🔸 **C Drive** (`C:\`)
- Program Files (various subfolders)
- PerfLogs, Users folders
- Look for: custom applications, user data

🔸 **print$ Share**
- Printer driver folders and files
- May contain uploaded malicious drivers

**🎯 Files to Look For:**
- `.txt`, `.log`, `.bak` files (potential sensitive data)
- `.exe`, `.bat`, `.ps1` files (potential malware upload locations)
- User folders (Desktop, Documents, Downloads)
- Database files (`.mdb`, `.sqlite`, `.db`)

**Related Scripts**
- `smb-enum-shares`: lists available shares (use **before** `smb-ls`)
- `smbclient`: allows file download/upload from SMB shares

---

## 📊 10. View SMB Server Stats
```bash
nmap -p 445 --script smb-server-stats <target-ip>
```
**📌 Purpose:**

This script gets real-time stats from the SMB server, like:
- How many files are open
- How many users are connected
- How many active SMB sessions
- Server uptime (if available)

**🧠 Why is this useful?**

This script helps monitor real-time usage of the SMB server. Useful for:
- 🔍 Spotting user or attacker activity (file access, logins)
- 🧪 Checking if the server is actively being used
- 📈 Monitoring for anomalies (like excessive failed logins or open files)

**INE Lab Example:**
```bash
nmap -p445 --script smb-server-stats --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
**📸 Sample Output:**
```bash
Host script results:
| smb-server-stats: 
|   Server statistics collected since 2025-09-09T13:33:45 (4m19s):
|     2250 bytes (8.69 b/s) sent, 2141 bytes (8.27 b/s) received
|_    0 failed logins, 0 permission errors, 0 system errors, 0 print jobs, 2 files opened
```
**Interpretation:**
- 🕓 **Uptime Info**: Stats collected since `2025-09-09T13:33:45`, running for `4 minutes and 19 seconds`
- 📤 **Data Sent**: 2250 bytes (~8.69 bytes/sec)
- 📥 **Data Received**: 2141 bytes (~8.27 bytes/sec)
- ❌ **Failed Logins**: 0 → No login errors occurred
- 🚫 **Permission Errors**: 0 → No unauthorized access attempts
- ⚙️ **System Errors**: 0 → No critical system-level errors
- 🖨️ **Print Jobs**: 0 → No activity related to print services
- 📁 **Files Opened**: 2 → Two files are currently open

**🚨 Red Flags to Watch For:**
- High number of failed logins → Possible brute force attack
- Many permission errors → Unauthorized access attempts
- Excessive open files → Potential data exfiltration

---

## 🧾 11. Using `--script-args` in Nmap
```bash
nmap -p 445 --script <script-name> --script-args <key>=<value>
```
**📌 Purpose:**

`--script-args` lets you customize the behavior of NSE scripts in Nmap by providing extra input (like usernames, passwords, timeouts, etc.).

It's like giving the script extra instructions.

### 🧪 Examples with SMB:
**🔐 Example 1:** Login with username and password
```bash
nmap -p 445 --script smb-enum-users --script-args smbuser=admin,smbpass=123456 <target-ip>
```

**🕵️‍♀️ Example 2:** Specify domain
```bash
nmap -p 445 --script smb-enum-shares --script-args smbuser=user1,smbpass=pass123,smbdomain=WORKGROUP <target-ip>
```

**🔄 Example 3:** Set script timeout
```bash
nmap -p 445 --script smb-enum-sessions --script-args timeout=20s <target-ip>
```

**🎭 Example 4:** Try anonymous access first
```bash
nmap -p 445 --script smb-enum-shares --script-args smbuser=guest,smbpass= <target-ip>
```

### 🔧 Common --script-args keys for SMB:
| Key          | Description                    | Example Values           |
|------------- |--------------------------------|--------------------------|
| `smbuser`    | Username for SMB login         | `administrator`, `guest` |
| `smbpass`    | Password for SMB login         | `password123`, ` ` (empty) |
| `smbdomain`  | Windows domain or workgroup    | `WORKGROUP`, `CORP.LOCAL` |
| `timeout`    | Script timeout (e.g. 10s)     | `10s`, `30s`, `1m`       |
| `smbnoguest` | Disable guest account fallback | `true`, `false`          |

### 💡 Pro Tips for script-args:
- Always try guest access first: `smbuser=guest,smbpass=`
- Use longer timeouts on slow networks: `timeout=60s`
- Specify domain for domain-joined machines: `smbdomain=CORP`

---

## 🔁 All-in-One Scan
```bash
# Basic scan (no authentication)
nmap -p 445 --script "smb-os-discovery,smb-protocols,smb-security-mode" <target-ip>

# Authenticated scan (with credentials)
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats" --script-args smbuser=administrator,smbpass=password123 <target-ip>

# Anonymous scan (guest access)
nmap -p 445 --script "smb-enum-shares,smb-ls" --script-args smbuser=guest,smbpass= <target-ip>
```

### 📋 Suggested Scan Order:
1. **Reconnaissance**: `smb-os-discovery`, `smb-protocols`, `smb-security-mode`
2. **Anonymous Testing**: `smb-enum-shares` with guest credentials
3. **Authenticated Enumeration**: All `smb-enum-*` scripts with valid credentials
4. **Deep Analysis**: `smb-ls`, `smb-server-stats` for detailed information

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
| **Host Unreachable** | No response | Check network connectivity and firewall rules |

### 🔍 Debugging Commands:
```bash
# Test basic connectivity
ping <target-ip>

# Check if SMB port is open
nmap -p 445 <target-ip>

# Test with verbose output
nmap -p 445 --script smb-enum-shares -v <target-ip>

# Try different authentication methods
nmap -p 445 --script smb-security-mode <target-ip>
```

### 📝 Common Error Messages:
- `NT_STATUS_ACCESS_DENIED` → Need valid credentials
- `NT_STATUS_LOGON_FAILURE` → Wrong username/password
- `NT_STATUS_ACCOUNT_DISABLED` → Account is disabled
- `NT_STATUS_PASSWORD_EXPIRED` → Password needs reset
- `NT_STATUS_NETWORK_UNREACHABLE` → Network/firewall issue
- `NT_STATUS_CONNECTION_REFUSED` → SMB service not running

### 🚀 Performance Optimization:
```bash
# Faster scanning with timing templates
nmap -p 445 --script smb-enum-shares -T4 <target-ip>

# Parallel script execution
nmap -p 445 --script "smb-enum-shares,smb-enum-users" --script-args timeout=10s <target-ip>

# Reduce unnecessary output
nmap -p 445 --script smb-enum-shares --script-args smbuser=guest,smbpass= <target-ip> 2>/dev/null
```

---

## ⚠️ Security Considerations

### 🔒 Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **Follow responsible disclosure** if vulnerabilities are found
- **Respect system availability** - don't crash production systems

### 🕵️ Operational Security (OpSec):
- **Logging Awareness**: These commands generate logs on target systems
- **Network Detection**: SMB enumeration can trigger IDS/IPS alerts
- **Timing**: Consider using slower scan rates (`-T1`, `-T2`) to avoid detection
- **VPN/Proxy**: Consider using anonymization tools in authorized tests

### 🚨 Detection Indicators:
Target systems may detect SMB enumeration through:
- **Event Logs**: Windows Security logs (Event ID 4624, 4625, 4648)
- **Network Monitoring**: Unusual SMB traffic patterns
- **Failed Login Attempts**: Multiple authentication failures
- **Process Monitoring**: SMB-related process spawning

### 🛡️ Defensive Countermeasures:
Organizations can protect against SMB enumeration by:
- **Disabling SMBv1**: Remove legacy protocol support
- **Enabling SMB Signing**: Require message signing
- **Network Segmentation**: Limit SMB access between subnets
- **Account Policies**: Enable account lockout, disable guest accounts
- **Monitoring**: Deploy SIEM rules for suspicious SMB activity

### 📊 Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| SMBv1 Enabled | 🔴 Critical | Disable immediately |
| Guest Account Active | 🟡 Medium | Disable or restrict |
| No Message Signing | 🟠 High | Enable signing requirement |
| Writable Admin Shares | 🔴 Critical | Review share permissions |
| Weak Passwords | 🟠 High | Implement password policy |

---

## 🔗 Related Tools and Scripts

### 🛠️ Complementary Tools:
```bash
# enum4linux - Comprehensive SMB enumeration
enum4linux -a <target-ip>

# smbclient - Interactive SMB client
smbclient -L <target-ip> -U username%password

# crackmapexec - Advanced SMB enumeration and exploitation
crackmapexec smb <target-ip> -u username -p password --shares

# impacket smbclient.py - Python SMB client
smbclient.py domain/username:password@<target-ip>

# rpcclient - RPC client for Windows
rpcclient -U username <target-ip>
```

### 📚 Additional NSE Scripts:
```bash
# Vulnerability scanning
nmap --script smb-vuln-* <target-ip>

# SMB2 specific scripts
nmap --script smb2-* <target-ip>

# Print spooler enumeration
nmap --script smb-print-text <target-ip>
```

### 🎯 Next Steps After SMB Enumeration:
1. **Password Attacks**: Use discovered usernames for brute-force
2. **Share Analysis**: Download and analyze accessible files
3. **Vulnerability Scanning**: Check for SMB-specific CVEs
4. **Lateral Movement**: Use SMB for post-exploitation activities
5. **Privilege Escalation**: Exploit misconfigurations found

---

## 📖 References and Further Reading

### 📚 Official Documentation:
- [Nmap NSE Scripts](https://nmap.org/nsedoc/scripts/) - Complete NSE script reference
- [Microsoft SMB Documentation](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) - Official SMB protocol docs

### 🎓 Learning Resources:
- [SANS SMB Enumeration Guide](https://www.sans.org/) - Professional pentesting techniques
- [Offensive Security SMB Attacks](https://www.offensive-security.com/) - Advanced SMB exploitation
- [HackTricks SMB Enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) - Comprehensive SMB testing guide

### 🔐 Security Research:
- [MS17-010 EternalBlue](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144) - Critical SMBv1 vulnerability
- [PrintNightmare](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-34527) - Print Spooler exploitation
- [SMB Relay Attacks](https://en.hackndo.com/ntlm-relay/) - Advanced SMB attack techniques

---

## 📋 Quick Reference Card

### 🚀 Most Useful Commands:
```bash
# Quick recon (no auth required)
nmap -p 445 --script smb-os-discovery,smb-security-mode <target>

# Guest access test
nmap -p 445 --script smb-enum-shares --script-args smbuser=guest,smbpass= <target>

# Full authenticated scan
nmap -p 445 --script "smb-enum-*" --script-args smbuser=admin,smbpass=pass <target>

# Check for vulnerabilities
nmap -p 445 --script smb-vuln-* <target>
```

### 🎯 Target Priority:
1. **High Value**: Servers, Domain Controllers, File Servers
2. **Medium Value**: Workstations with admin users
3. **Low Value**: Guest systems, isolated machines

### ⏱️ Time Estimates:
- **Basic enumeration**: 2-5 minutes
- **Full authenticated scan**: 5-15 minutes  
- **Deep file analysis**: 15+ minutes

---

**Last Updated**: September 2025  
**Version**: 2.1  
**Author**: Penetration Testing Team
