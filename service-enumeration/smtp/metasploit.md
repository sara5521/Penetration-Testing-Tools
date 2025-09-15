# 📧 Metasploit - SMTP Enumeration Modules

This file explains useful **Metasploit auxiliary modules** for SMTP service enumeration.

---

## 🎯 Purpose

These modules help you:
- Detect SMTP server software and version  
- Enumerate valid SMTP users (VRFY/EXPN/RCPT checks)  
- Check for open relay behavior and relay abuse possibilities  
- Enumerate NTLM domain information via SMTP (if supported)  
- Aggregate findings for follow-up credential or configuration testing  

---

## 🗂️ SMTP Enumeration Modules Summary

| #  | Module                                 | What it Does                                      |
|----|----------------------------------------|---------------------------------------------------|
| 1  | auxiliary/scanner/smtp/smtp_version    | Detect SMTP server software/version (banner)      |
| 2  | auxiliary/scanner/smtp/smtp_enum       | Enumerate users via VRFY/EXPN/RCPT methods        |
| 3  | auxiliary/scanner/smtp/smtp_relay      | Test for open relay / relay acceptance behavior   |
| 4  | auxiliary/scanner/smtp/smtp_ntlm_domain| Attempt to enumerate NTLM/Windows domain via SMTP |
| 5  | auxiliary/scanner/smtp/smtp_vrfy       | Focused VRFY checks (if present in framework)     |

---

## 1️⃣ Detect SMTP Version / Banner

**Module**
```bash
auxiliary/scanner/smtp/smtp_version
```

**Command**
```bash
use auxiliary/scanner/smtp/smtp_version
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:**
```
[*] 192.253.225.3:25 - Banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
[*] demo.ine.local:25 - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

🔍 **Interpretation:**
- Banner reveals **Postfix** as the MTA and the hostname `openmailbox.xyz`.
- Useful to pick version-specific checks or exploits and understand mail routing.
- If STARTTLS is advertised, consider using encrypted sessions for further manual checks.

---

## 2️⃣ SMTP User Enumeration (smtp_enum)

**Module**
```bash
auxiliary/scanner/smtp/smtp_enum
```

**Command**
```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS demo.ine.local
set USER_FILE /path/to/userlist.txt
set THREADS 5             # optional: speed up enumeration
run
```

📸 **Sample Output:** (combined from lab outputs)
```
[*] 192.253.225.3:25 - 192.253.225.3:25 Banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
[+] 192.253.225.3:25 - Users found: _apt, admin, administrator, backup, bin, daemon, games, gnats, irc, list, lp, mail, man, news, nobody, postfix, postmaster, proxy, sync, sys, uucp, www-data
[*] demo.ine.local:25 - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

🔍 **Interpretation:**
- Discovers **system/service accounts** and common admin accounts (e.g., `admin`, `postmaster`, `www-data`).
- An empty entry in results can be an artifact; re-check with a focused test.
- Correlate with `smtp-user-enum` and manual `VRFY` to validate results.

### 📂 Next Steps After Finding Users
- Try `RCPT TO` checks or cautious password guessing against allowed auth mechanisms (only on authorized engagements).
- Note service accounts like `www-data` — useful for web-related post-exploitation paths.

---

## 3️⃣ SMTP Open Relay Testing (smtp_relay)

**Module**
```bash
auxiliary/scanner/smtp/smtp_relay
```

**Command**
```bash
use auxiliary/scanner/smtp/smtp_relay
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:** (example)
```
[*] 192.253.225.3:25 - Testing relay for a list of combinations...
[!] 192.253.225.3:25 - Server accepted relay for <attacker@evil.com> -> <victim@external.com> (OPEN RELAY)
[*] demo.ine.local:25 - Scanned 1 of 1 hosts (100% complete)
```

🔍 **Interpretation:**
- If the server accepts relay for arbitrary external recipients → it's an **open relay** and can be abused for spam or phishing.
- Relay findings are high-impact; report immediately to client/owner and avoid abusing the relay.

---

## 4️⃣ SMTP NTLM / Domain Enumeration (smtp_ntlm_domain)

**Module**
```bash
auxiliary/scanner/smtp/smtp_ntlm_domain
```

**Command**
```bash
use auxiliary/scanner/smtp/smtp_ntlm_domain
set RHOSTS demo.ine.local
run
```

📸 **Sample Output:** (example)
```
[*] 192.253.225.3:25 - Attempting NTLM domain discovery...
[+] 192.253.225.3:25 - Domain: EXISTSE.LOCAL, Domain Controller: dc1.existse.local
[*] demo.ine.local:25 - Scanned 1 of 1 hosts (100% complete)
```

🔍 **Interpretation:**
- If successful, this module can reveal **Windows domain** information which aids lateral movement planning.
- Use results to correlate with SMB/LDAP enumeration for more domain context.

---

## 5️⃣ Focused VRFY Checks (smtp_vrfy)

**Note:** Some Metasploit installs provide focused VRFY/EXPN helpers; otherwise `smtp_enum` handles those checks. If `smtp_vrfy` exists in your framework, you can run focused VRFY tests as shown below.

**Command Example (manual VRFY via Telnet)**
```bash
telnet demo.ine.local 25
HELO attacker.xyz
EHLO attacker.xyz
VRFY www-data
VRFY admin
```

✅ **Response Example:**
```
252 2.0.0 <www-data> exists
250 2.1.5 <admin> exists
```

🔍 **Interpretation:**
- `VRFY` confirms if a username exists; `EXPN` may expand mailing lists to show members.
- If VRFY is disabled, consider `RCPT TO` checks during an SMTP transaction.

---

## 🔎 Key Findings (from lab examples)

- **Banner:** `220 openmailbox.xyz ESMTP Postfix` — identifies the mail server software.  
- **Discovered users:** `_apt, admin, administrator, backup, bin, daemon, games, gnats, irc, list, lp, mail, man, news, nobody, postfix, postmaster, proxy, sync, sys, uucp, www-data` — mix of system and admin/service accounts.  
- **High-impact issues to look for:** open relay, exposed admin users, domain info leakage via NTLM.

---

## ⚠️ Notes & Best Practices

- **Validate with multiple tools:** Cross-check Metasploit results with `smtp-user-enum` and manual `telnet`/`openssl s_client` checks.  
- **Use conservative threading/timeouts:** `set THREADS` can speed scans but may trigger IDS/IPS or cause stability issues.  
- **Respect rules of engagement:** Only test systems you are authorized to scan. SMTP tests (relay checks, VRFY) can be disruptive or trigger blacklisting.  
- **Log and store loot:** Metasploit stores outputs in loot files under `~/.msf4/loot/`; keep these for reporting.  

---

## ✅ Recommended file name & location in your repo

```
service-enumeration/smtp/metasploit.md
```

---

## 🔁 Suggested Next Steps
- Cross-check usernames with `smtp-user-enum` results and manual VRFY tests.
- If domain info found, continue with SMB/LDAP enumeration under `service-enumeration/smb` and `service-enumeration/ldap`.
- Add any validated usernames to your credential list for safe, authorized authentication testing later.
