# Nmap - SMB Enumeration (Overview)

Path: `service-enumeration/smb/`

Purpose
This overview lists the Nmap NSE scripts used to enumerate SMB. Use it as entry point. Each script has a dedicated file with detailed usage, examples, interpretation, caveats and remediation steps.

Files in this folder
- smb-os-discovery.md
- smb-protocols.md
- smb-security-mode.md
- smb-enum-domains.md
- smb-enum-users.md
- smb-enum-groups.md
- smb-enum-sessions.md
- smb-enum-services.md
- smb-enum-shares.md
- smb-ls.md
- smb-server-stats.md
- script-args.md

Quick command patterns
- Single script: `nmap -p445 --script <script-name> <target>`
- Authenticated: `--script-args 'smbuser=<user>,smbpass=<pass>'`
- Save output: `-oA smb-recon`

Recommended workflow
1. smb-os-discovery
2. smb-protocols
3. smb-security-mode
4. Run authenticated scripts if you have credentials: smb-enum-*
5. smb-enum-shares → smb-ls for contents
6. smb-server-stats for runtime context

Evidence
Save raw output and XML: `nmap -oA smb-recon ...`.


# smb-os-discovery

**Command**
```
nmap -p445 --script smb-os-discovery <target>
```

**Purpose**
Probe SMB to retrieve OS string, NetBIOS name, domain/FQDN, and system time.

**INE Lab Example**
```
nmap --script smb-os-discovery -p 445 demo.ine.local
```

**Sample Output**
```
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: demo
|   NetBIOS computer name: SAMBA-RECON
|   Domain name: ine.local
|   FQDN: demo.ine.local
|   System time: 2024-07-05T01:34:01+08:00
```

**Interpretation**
- Reveals OS family and Samba/Windows version.
- Provides hostname, NetBIOS and domain for mapping.

**Caveats**
- Banners may be forged or limited by permissions.
- Some fields require authenticated queries for full detail.

**Follow-ups**
- Run `smb-protocols` and `smb-security-mode` next.
- Search CVE records for detected server string.

**Remediation**
- Keep SMB up to date and avoid exposing SMB to untrusted networks.


# smb-protocols

**Command**
```
nmap -p445 --script smb-protocols <target>
```

**Purpose**
Enumerate supported SMB dialects and protocol features.

**INE Lab Example**
```
nmap -p445 --script smb-protocols demo.ine.local
```

**Sample Output**
```
| smb-protocols:
|   dialects: SMBv1, SMBv2, SMBv3
|   dialects2: dialect 3.1.1
```

**Interpretation**
- Shows whether SMBv1 is enabled which is high risk.
- Shows dialect details for negotiation behavior.

**Caveats**
- Middleboxes may affect dialect negotiation.
- Some servers negotiate different dialects per client.

**Follow-ups**
- If SMBv1 appears, run `smb-vuln-ms17-010` and schedule patching.
- Use smbclient to validate behavior.

**Remediation**
- Disable SMBv1, enable/require modern dialects and patch OS.


# smb-security-mode

**Command**
```
nmap -p445 --script smb-security-mode <target>
```

**Purpose**
Check message signing, authentication level and anonymous/guest access.

**INE Lab Example**
```
nmap -p445 --script smb-security-mode demo.ine.local
```

**Sample Output**
```
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled
```

**Interpretation**
- `message_signing: disabled` indicates MITM risk.
- `account_used` shows whether result came from guest/anon access.

**Caveats**
- Script output depends on access level; unauthenticated results may be incomplete.

**Follow-ups**
- If signing disabled, test with authenticated credentials to confirm.
- Correlate with group policy on domain controllers.

**Remediation**
- Require SMB signing via group policy. Disable guest access where not needed.


# smb-enum-domains

**Command**
```
nmap -p445 --script smb-enum-domains --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
Enumerate domain/workgroup names, basic user lists and password policy hints.

**INE Lab Example**
```
nmap -p445 --script smb-enum-domains --script-args smbuser=administrator,smbpass=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-domains:
|  WIN-OMCNBKR66MN
|    Users: Administrator, bob, Guest
|    Passwords: max age: 42 days; complexity: enabled; lockout: disabled
```

**Interpretation**
- Reveals domain name and some password policy attributes.
- Lockout disabled makes online brute-force easier.

**Caveats**
- Requires authentication for full detail.
- Anonymous scan may return partial data.

**Follow-ups**
- Use the domain name for LDAP/AD follow-ups and mapping.

**Remediation**
- Enforce account lockout and strong password policies; limit who can query domain data.


# smb-enum-users

**Command**
```
nmap -p445 --script smb-enum-users --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
List user accounts and RIDs. Identify accounts with special flags.

**INE Lab Example**
```
nmap -p445 --script smb-enum-users --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-users:
|   WIN-...\Administrator (RID: 500) - Password does not expire
|   WIN-...\bob (RID: 1010)
|_  WIN-...\Guest (RID: 501) - Password not required
```

**Interpretation**
- Administrator (RID 500) is built-in admin.
- Guest with no password is high risk.

**Caveats**
- Modern systems restrict enumeration; results may be incomplete.
- False positives possible if accounts renamed.

**Follow-ups**
- Add validated usernames to credential lists for controlled testing.
- Cross-check with LDAP, web app users, and repos.

**Remediation**
- Disable Guest or require passwords; audit admin accounts; log enumeration attempts.


# smb-enum-groups

**Command**
```
nmap -p445 --script smb-enum-groups --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
List groups and members to identify privilege boundaries.

**INE Lab Example**
```
nmap -p445 --script smb-enum-groups --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-groups:
|  Builtin\Administrators: Administrator, bob
|  Builtin\Users: bob
|_ Builtin\Remote Desktop Users: bob
```

**Interpretation**
- Shows which users have Admin or RDP privileges.

**Caveats**
- May require privileges to list full membership.

**Follow-ups**
- Use group info to prioritize accounts for testing and lateral movement.

**Remediation**
- Reduce unnecessary admin memberships and apply least privilege.


# smb-enum-sessions

**Command**
```
nmap -p445 --script smb-enum-sessions --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
List currently active SMB sessions and source hosts.

**INE Lab Example**
```
nmap -p445 --script smb-enum-sessions --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-sessions:
|   Users logged in: bob since 2025-09-09T09:43:25
|_  ADMINISTRATOR connected from \10.10.45.4
```

**Interpretation**
- Real-time info on logged-in users and source IPs.

**Caveats**
- Data is ephemeral and may change quickly.

**Follow-ups**
- Correlate session info with logs and avoid interfering with active users.

**Remediation**
- Monitor and investigate unexpected sessions; enforce session timeouts.


# smb-enum-services

**Command**
```
nmap -p445 --script smb-enum-services --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
Enumerate Windows services and their state via Service Control Manager calls.

**INE Lab Example**
```
nmap -p445 --script smb-enum-services --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-services:
|  Spooler: SERVICE_RUNNING
|  AmazonSSMAgent: SERVICE_RUNNING
|_ TrustedInstaller: SERVICE_RUNNING
```

**Interpretation**
- Identifies running services like Spooler which have known exploit history.

**Caveats**
- Requires permissions to list some services.

**Follow-ups**
- Inspect service binary paths, startup accounts and permissions as follow-ups.

**Remediation**
- Patch vulnerable services and restrict service installation privileges.


# smb-enum-shares

**Command**
```
nmap -p445 --script smb-enum-shares --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
Discover SMB shares, remarks, and access rights (read/write), including hidden admin shares.

**INE Lab Example**
```
nmap -p445 --script smb-enum-shares --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-enum-shares:
|  ADMIN$ -> Hidden admin share (C:\Windows) [READ/WRITE]
|  C$ -> Hidden root drive [READ/WRITE]
|  Documents -> Admin Documents [READ]
|_ Downloads -> Admin Downloads [READ]
```

**Interpretation**
- Reveals ADMIN$, C$, IPC$ and user-created shares and permissions.

**Caveats**
- Writable admin shares are high-risk for file manipulation.

**Follow-ups**
- Use smbclient or smbget to pull files for offline analysis if permitted.

**Remediation**
- Remove secrets from shares and restrict admin-share access.


# smb-ls

**Command**
```
nmap -p445 --script smb-ls --script-args 'smbuser=admin,smbpass=pass,share=Documents' <target>
```

**Purpose**
List files and folders inside an SMB share. Used after smb-enum-shares.

**INE Lab Example**
```
nmap -p445 --script smb-enum-shares,smb-ls --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
\demo\Documents\report.docx
\demo\Downloads\setup.exe
\demo\ADMIN$\ADFS\...
```

**Interpretation**
- Useful to locate backups, config files, keys, flags.

**Caveats**
- Large shares can be noisy and slow; scope to specific paths where possible.

**Follow-ups**
- If allowed, download suspicious files and analyze in an isolated environment.

**Remediation**
- Enforce ACLs on sensitive share folders and remove unnecessary files.


# smb-server-stats

**Command**
```
nmap -p445 --script smb-server-stats --script-args 'smbuser=admin,smbpass=pass' <target>
```

**Purpose**
Collect runtime statistics: bytes transferred, open files, failed logins, uptime window.

**INE Lab Example**
```
nmap -p445 --script smb-server-stats --script-args smbusername=administrator,smbpassword=smbserver_771 demo.ine.local
```

**Sample Output**
```
| smb-server-stats:
|   Server statistics collected since 2025-09-09T13:33:45:
|    2250 bytes sent, 2141 bytes received
|    0 failed logins, 2 files opened
|_   0 permission errors
```

**Interpretation**
- Snapshot of server load and errors; failed logins indicate attacks.

**Caveats**
- Stats are transient; capture timestamps and correlate with logs.

**Follow-ups**
- Use stats to identify anomalies and timing for follow-ups.

**Remediation**
- Configure monitoring and alerting on failed login thresholds and spikes.


# NSE script arguments for SMB

Purpose
Describe common `--script-args` keys used by SMB NSE scripts and examples.

Common keys
- smbuser: username for auth
- smbpass: password for auth
- domain: domain or workgroup name
- share: share name (for smb-ls)
- timeout: script timeout (e.g. 20s)

Examples
- Authenticated users list:
```
nmap -p445 --script smb-enum-users --script-args 'smbuser=administrator,smbpass=Passw0rd' <target>
```
- Specify domain and share:
```
nmap -p445 --script smb-enum-shares --script-args 'smbuser=user1,smbpass=pass123,domain=WORKGROUP' <target>
```
- Set timeout:
```
nmap -p445 --script smb-enum-sessions --script-args 'timeout=20s' <target>
```

Tips
- Use single quotes around --script-args to avoid shell expansion.
- Prefer least-privileged credentials for authenticated scans.
- Save output with `-oA` for evidence.


