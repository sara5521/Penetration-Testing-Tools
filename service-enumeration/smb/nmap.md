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

## 🗂️ SMB Enumeration Scripts Summary

|   | Script                  | What it Does                             |
|---|-------------------------|-------------------------------------------|
| 1 | smb-os-discovery       | Detect OS and hostname                   |
| 2️ | smb-protocols          | Show SMB versions supported              |
| 3️ | smb-security-mode      | Check SMB signing and auth level         |
| 4️ | smb-enum-domains       | List domain/workgroup info               |
| 5️ | smb-enum-users         | List user accounts                       |
| 6️ | smb-enum-groups        | List Windows groups                      |
| 7️ | smb-enum-sessions      | List active SMB sessions                 |
| 8️ | smb-enum-services      | List running services                    |
| 9️ | smb-enum-shares        | List shared folders                      |
| 10 | smb-ls                | List files inside shares                 |
| 11 | smb-server-stats      | Show stats like open files/sessions      |
| 1️2️ | --script-args         | Customize script input (username, etc.)  |

---

## 🔧 Common SMB Nmap Scripts

## 🖥️ 1. Discover OS and Hostname
```bash
nmap -p 445 --script smb-os-discovery <target-ip>
```
---

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

---

## 🔐 3. Check SMB Security Mode
```bash
nmap -p 445 --script smb-security-mode <target-ip>
```
#### 📌 Purpose:
This script checks how secure the SMB server is by showing:
- If SMB signing is supported
- If SMB signing is required
- What type of authentication is used (user or share level)

#### 🧠 Why is this useful?
- If signing is not required, the server might be vulnerable to spoofing or MiTM (man-in-the-middle) attacks.
- Helps you understand how the target is protected.

#### INE Lab Example:
``` bash
nmap -p445 --script smb-security-mode demo.ine.local
```
📸 Sample Output:
```bash
Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```
#### Interpretation:
- 🔑 Account Used: Guest (anonymous)
- 🛡️ Auth Level: User-level (safer than share-level)
- 🔁 Challenge/Response: Supported (adds some protection)
- ⚠️ Message Signing: Disabled (vulnerable to MITM attacks)
  
---

## 🏢 4. Enumerate SMB Domains / Workgroups
```bash
nmap -p 445 --script smb-enum-domains <target-ip>
```
#### 📌 Purpose:
This script tries to list the Windows domains or workgroups the SMB server belongs to.

Think of a domain as the "name of the Windows network" (like `CORP`, `DEMO.LOCAL`, or `WORKGROUP`).
#### 🧠 Why is this useful?
- Helps you know what domain or workgroup the machine belongs to.
- Useful for:
  - Active Directory attacks
  - Targeting specific domain users
  - Mapping the Windows network
- Sometimes reveals the domain SID (Security Identifier)

#### INE Lab Example:
``` bash
nmap -p445 --script smb-enum-domains --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
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

#### Interpretation:
- 💻 Domain name: `WIN-OMCNBKR66MN`
- 👥 Users: Administrator, bob, Guest
- 👮‍♂️ Groups: WinRMRemoteWMIUsers__
- ⏱️ Password Policy:
  - Max age = 42 days
  - Complexity = Enabled
  - Lockout = Disabled
- 🧱 Builtin Groups: Show default permission roles like Admins, Guests, RDP users, etc.
- 🕓 Domain created: 2013-08-22

---

## 👥 5. Enumerate Users
```bash
nmap -p 445 --script smb-enum-users <target-ip>
```
#### 📌 Purpose:
This script tries to list user accounts on a Windows machine by querying the SMB service.
#### 🧠 Why is this useful?
- It gives you a list of usernames that exist on the system.
- You can use these names for:
  - Brute-force attacks (e.g. with Hydra)
  - Password spraying
  - Understanding roles and access levels
- Sometimes even works with anonymous access

#### INE Lab Example:
``` bash
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
#### Interpretation:
- 🔐 Administrator is the default admin account — useful for brute-force attacks.
- 👨‍💻 bob is likely a regular user account added by an admin — try brute-forcing it.
- 🧳 Guest has:
  - No password required ❗
  - Useful for anonymous logins and privilege escalation
- 🔢 RIDs (Relative IDs):
  - 500 = Administrator
  - 501 = Guest
  - 1000 = Custom user accounts

---

## 👤 6. Enumerate User Groups
```bash
nmap -p 445 --script smb-enum-groups <target-ip>
```
#### 📌 Purpose:
This script tries to list the Windows groups on the target — groups are like "roles" or "permission levels" for users.
#### 🧠 Why is this useful?
- Helps you understand user roles and privileges.
- Can reveal if there’s a “Remote Desktop Users” group (target for RDP access).
- Useful for:
  - Privilege escalation
  - Brute-force targeting
  - Mapping group memberships in Active Directory

#### INE Lab Example:
``` bash
nmap -p445 --script smb-enum-groups --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
``` bash
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
#### Interpretation:
- `Administrator` and `bob` are members of the Administrators group → they have full system privileges.
- `bob` is also part of the Remote Desktop Users group → this means he may log in via RDP (Remote Desktop Protocol), which can be a target for brute-force attacks.
- `Guest` is in the Guests group → usually has limited permissions and is less useful for privilege escalation.
- Other groups like `Backup Operators`, `IIS_IUSRS`, etc., are empty → you can ignore them unless you find users assigned to them later.

💡 **TIP**: This group membership information is useful for:
- **Privilege escalation** — targeting users with high privileges.
- **Lateral movement** — if RDP or other remote services are enabled.
- **Brute-force attacks** — focus on privileged accounts like ```bob```.

---

## 🔄 7. List Active Sessions
```bash
nmap -p 445 --script smb-enum-sessions <target-ip>
```
#### 📌 Purpose:
This script tries to list current SMB sessions — in other words, it shows:
- Who is currently connected to the SMB server
- What users or machines are active
- How many open sessions exist
#### 🧠 Why is this useful?
- Shows real-time activity on the target.
- Helps identify:
  - Logged-in users
  - IPs of other attackers or admins
  - If the system is actively being used

#### INE Lab Example:
``` bash
nmap -p445 --script smb-enum-sessions --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```
📸 Sample Output:
``` bash
Host script results:
| smb-enum-sessions: 
|   Users logged in
|     WIN-OMCNBKR66MN\bob since 2025-09-09T09:43:25
|   Active SMB sessions
|_    ADMINISTRATOR is connected from \\10.10.45.4 for [just logged in, it's probably you], idle for [not idle]
```
#### Interpretation:
- 👤 bob is currently logged into the system.
- ⏱️ His session has been active since 09:43:25.
- 🧑‍💼 The Administrator account is also logged in — likely you.
- 🖥️ That session came from IP 10.10.45.4.
- 🔄 It is currently active, not idle.

---

## ⚙️ 8. Enumerate Running Services
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
#### 🧠 Why is this useful?
- Shows which services are running or stopped.
- Helps identify possible entry points or privilege escalation targets.
- Example: If `Remote Desktop` is running, you might try RDP login later.

---

## 📂 9. Enumerate and Browse SMB Shares

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
#### 🧠 Why is this useful?
- Helps you find files or folders that are publicly accessible
- Great for finding sensitive data like `flag.txt`, `backup.zip`, or `config.bak`.
- If a share is writable, you might be able to upload malicious files.

### List files inside the shares:
```bash
nmap -p 445 --script smb-ls <target-ip>
```
#### 📌 Purpose:
This script tries to list the actual files and folders inside each shared folder (share) found on the SMB server.

So after you discover shares with `smb-enum-shares`, you can use `smb-ls` to look inside them.

In other words, use `smb-enum-shares` to identify available shares, and `smb-ls` to see the contents inside them.
#### 🧠 Why is this useful?
- Helps you see real files in public shares.
- You may find:
  - `flag.txt`
  - `credentials.txt`
  - `backup.zip`
- If you find something interesting, you can download it using `smbclient`.

---

## 📊 10. View SMB Server Stats
```bash
nmap -p 445 --script smb-server-stats <target-ip>
```
#### 📌 Purpose:
This script gets real-time stats from the SMB server, like:
- How many files are open
- How many users are connected
- How many active SMB sessions
- Server uptime (if available)
#### 🧠 Why is this useful?
- Shows if the server is busy or being used by other people.
- Helps you find out:
  - If an admin or another hacker is connected 👀
  - How long the server has been running
- Can reveal useful behavior or targets for privilege escalation

---

## 🧾 11. Using `--script-args` in Nmap
```bash
nmap -p 445 --script <script-name> --script-args <key>=<value>
```
#### 📌 Purpose:
`--script-args` lets you customize the behavior of NSE scripts in Nmap by providing extra input (like usernames, passwords, timeouts, etc.).

It’s like giving the script extra instructions.

### 🧪 Examples with SMB:
#### 🔐 Example 1: Login with username and password
```bash
nmap -p 445 --script smb-enum-users --script-args smbuser=admin,smbpass=123456 <target-ip>
```

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

---

## 🔁 All-in-One Scan
```bash
nmap -p 445 --script "smb-enum-*,smb-os-discovery,smb-security-mode,smb-server-stats" <target-ip>
```

---

## 🧠 Tips
- Make sure port 445 is open.
- Some scripts may require authentication.
- Combine with tools like:
  - `enum4linux`
  - `smbclient`
  - `crackmapexec`

---

📖 [View on NSE Scripts](https://nmap.org/nsedoc/scripts)

