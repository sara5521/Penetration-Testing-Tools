# 📦 Nmap - SMB Enumeration Scripts

This section includes useful Nmap scripts to enumerate SMB (Server Message Block) services on Windows machines.

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

## 🔧 Common SMB Nmap Scripts

## 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```

## 🌐 2. Detect SMB Protocol Version
```bash
nmap -p 445 --script smb-protocols <target-ip>
```
#### 📌 Purpose:
This script shows which SMB version the server supports:
- SMBv1 → Old and not secure ⚠️
- SMBv2 / SMBv3 → Newer and more secure ✅
#### 🧠 Why is this useful?
- If the server supports SMBv1, it's risky and may be vulnerable to attacks like EternalBlue.
- Tells you whether modern, secure protocols (SMBv3) are supported.
- Important for deciding what kind of attack or enumeration you can try.

## 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```
#### 📌 Purpose:
This script checks how secure the SMB server is by showing:
- If SMB signing is supported
- If SMB signing is required
- What type of authentication is used (user or share level)
#### 🧠 Why it's useful?
- If signing is not required, the server might be vulnerable to spoofing or MiTM (man-in-the-middle) attacks.
- Helps you understand how the target is protected.

## 🔄 4. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```
#### 📌 Purpose:
This script tries to list current SMB sessions — in other words, it shows:
- Who is currently connected to the SMB server
- What users or machines are active
- How many open sessions exist
#### 🧠 Why it's useful?
- Shows real-time activity on the target.
- Helps identify:
  - Logged-in users
  - IPs of other attackers or admins
  - If the system is actively being used

## 📂 5. Enumerate and Browse SMB Shares

### List shared folders:
```bash
nmap -p 445 --script smb-enum-shares <target-ip>
```
#### 📌 Purpose:
This script lists all shared folders (also called shares) on a Windows machine over SMB.

It tells you:
- Which folders are being shared
- Whether they are readable or writable
- If they're administrative or public shares
#### 🧠 Why it's useful?
- Helps you find files or folders that are publicly accessible
- Great for finding sensitive data like ```flag.txt```, ```backup.zip```, or ```config.bak```.
- If a share is writable, you might be able to upload malicious files.

### List files inside the shares:
```bash
nmap -p 445 --script smb-ls <target-ip>
```
#### 📌 Purpose:
This script tries to list the actual files and folders inside each shared folder (share) found on the SMB server.

So after you discover shares with ```smb-enum-shares```, you can use ```smb-ls``` to look inside them.

In other words, use ```smb-enum-shares``` to identify available shares, and ```smb-ls``` to see the contents inside them.
#### 🧠 Why it's useful?
- Helps you see real files in public shares.
- You may find:
  - ```flag.txt```
  - ```credentials.txt```
  - ```backup.zip```
- If you find something interesting, you can download it using ```smbclient```.

## 👥 6. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```
#### 📌 Purpose:
This script tries to list user accounts on a Windows machine by querying the SMB service.
#### 🧠 Why it's useful?
- It gives you a list of usernames that exist on the system.
- You can use these names for:
  - Brute-force attacks (e.g. with Hydra)
  - Password spraying
  - Understanding roles and access levels
- Sometimes even works with anonymous access

## 📊 7. View SMB Server Stats
```bash
nmap -p 445 --script smb-server-stats <target-ip>
```
#### 📌 Purpose:
This script gets real-time stats from the SMB server, like:
- How many files are open
- How many users are connected
- How many active SMB sessions
- Server uptime (if available)
#### 🧠 Why it's useful?
- Shows if the server is busy or being used by other people.
- Helps you find out:
  - If an admin or another hacker is connected 👀
  - How long the server has been running
- Can reveal useful behavior or targets for privilege escalation

## 🏢 8. Enumerate SMB Domains / Workgroups
```bash
nmap -p 445 --script smb-enum-domains <target-ip>
```
#### 📌 Purpose:
This script tries to list the Windows domains or workgroups the SMB server belongs to.

Think of a domain as the "name of the Windows network" (like ```CORP```, ```DEMO.LOCAL```, or ```WORKGROUP```).
#### 🧠 Why it's useful?
- Helps you know what domain or workgroup the machine belongs to.
- Useful for:
  - Active Directory attacks
  - Targeting specific domain users
  - Mapping the Windows network
- Sometimes reveals the domain SID (Security Identifier)

## 👤 9. Enumerate User Groups
```bash
nmap -p 445 --script smb-enum-groups <target-ip>
```
#### 📌 Purpose:
This script tries to list the Windows groups on the target — groups are like "roles" or "permission levels" for users.
#### 🧠 Why it's useful?
- Helps you understand user roles and privileges.
- Can reveal if there’s a “Remote Desktop Users” group (target for RDP access).
- Useful for:
  - Privilege escalation
  - Brute-force targeting
  - Mapping group memberships in Active Directory

## ⚙️ 10. Enumerate Running Services
```bash
nmap -p 445 --script smb-enum-services <target-ip>
```
#### 📌 Purpose:
This script tries to list the Windows services that are running on the target through SMB.

Think of services as background programs like:
- Remote Desktop
- Windows Update
- Print Spooler
- Antivirus
- etc.
#### 🧠 Why it's useful?
- Shows which services are running or stopped.
- Helps identify possible entry points or privilege escalation targets.
- Example: If ```Remote Desktop``` is running, you might try RDP login later.

## 🧾 What is ```--script-args``` in Nmap?
```bash
nmap -p 445 --script-args
```
#### 📌 Purpose:
```--script-args``` lets you customize the behavior of NSE scripts in Nmap by providing extra input (like usernames, passwords, timeouts, etc.).

It’s like giving the script extra instructions.
#### ✅ Basic Syntax:
```bash
nmap -p 445 --script <script-name> --script-args <key>=<value>
```
You can provide one or more arguments, separated by commas.
### 🧪 Examples with SMB:
#### 🔐 Example 1: Login with username and password
```bash
nmap -p 445 --script smb-enum-users --script-args smbuser=admin,smbpass=123456 <target-ip>
```
This tells the script to try using these credentials instead of anonymous login.
#### 🕵️‍♀️ Example 2: Specify domain
```bash
nmap -p 445 --script smb-enum-shares --script-args smbuser=user1,smbpass=pass123,domain=WORKGROUP <target-ip>
```
#### 🔄 Example 3: Set script timeout
```bash
nmap -p 445 --script smb-enum-sessions --script-args timeout=20s <target-ip>
```
### 🔧 Common --script-args keys for SMB:
| Key       | Description                 |
| --------- | --------------------------- |
| `smbuser` | Username for SMB login      |
| `smbpass` | Password for SMB login      |
| `domain`  | Windows domain or workgroup |
| `timeout` | Script timeout (e.g. 10s)   |

## 🔁 All-in-One Scan
```bash
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats" <target-ip>
```

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts may require authentication.
- Combine with tools like:
  - ```enum4linux```
  - ```smbclient```
  - ```crackmapexec```
