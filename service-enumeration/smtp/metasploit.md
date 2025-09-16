# Metasploit - SMTP Enumeration Modules (Detailed)

**Path:** `service-enumeration/smtp/metasploit.md`

## Purpose
Detailed notes for Metasploit auxiliary modules used to enumerate SMTP services.  
Use this file to identify server software, find valid users, test relay behavior, discover domain info, and collect evidence for follow-up testing.

## Where to save
`service-enumeration/smtp/metasploit.md`

## Key modules / usage
- `auxiliary/scanner/smtp/smtp_version` — read banner and detect MTA.
- `auxiliary/scanner/smtp/smtp_enum` — enumerate users using VRFY/EXPN/RCPT checks.
- `auxiliary/scanner/smtp/smtp_relay` — test for open relay behavior.
- `auxiliary/scanner/smtp/smtp_ntlm_domain` — attempt NTLM domain discovery via SMTP.
- `auxiliary/scanner/smtp/smtp_vrfy` — focused VRFY checks (if present).
- CLI tools for manual checks: `telnet`, `openssl s_client`, `smtp-user-enum`.

---

## Quick examples
```bash
# Start msfconsole
msfconsole -q

# Banner / version
use auxiliary/scanner/smtp/smtp_version
set RHOSTS demo.ine.local
run

# User enumeration
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS demo.ine.local
set USER_FILE /path/to/users.txt
run

# Relay test
use auxiliary/scanner/smtp/smtp_relay
set RHOSTS demo.ine.local
run

# NTLM/domain discovery
use auxiliary/scanner/smtp/smtp_ntlm_domain
set RHOSTS demo.ine.local
run
```

---

## How to read common output
- `Banner: 220 ...` — MTA greeting. May include software and hostname.  
- `Users found:` — list of usernames discovered by enumeration. Validate before using.  
- `Server accepted relay` or similar — indicates open relay behavior. High severity.  
- `Domain:` — returned domain name or DC host. Useful for follow-up with SMB/LDAP.

---

## Practical tips
- Use `smtp_version` before other modules to tailor checks to the MTA (Postfix, Exim, Exchange).  
- Prefer non-destructive checks first (`smtp_version`, `smtp_enum` with low threads).  
- Cross-check results with `smtp-user-enum`, manual VRFY via `telnet`, and `openssl s_client -starttls smtp` where STARTTLS is available.  
- Log raw module output and timestamps (`~/.msf4/loot/` or spool).  
- Relay testing can cause spam; do not abuse. Report immediately if you find an open relay.

---

# Module details (expanded)

## auxiliary/scanner/smtp/smtp_version — Detect MTA and banner

**Purpose (short)**  
Read SMTP greeting banner and basic capabilities.

**Important options**
- `RHOSTS`, `RPORT` (25), `SSL` / `STARTTLS` handling, `THREADS`.

**What it does**
- Connects to SMTP port and reads the 220 greeting. Optionally issues EHLO to enumerate advertised extensions (STARTTLS, AUTH methods).

**Typical output**
```
[*] 192.253.225.3:25 - Banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

**Interpretation**
- Banner often reveals MTA name and sometimes version. EHLO output shows supported extensions and STARTTLS support.

**Caveats**
- Banners can be customized or hidden. Rely on multiple checks.

**Follow-ups**
- If STARTTLS advertised, use `openssl s_client -starttls smtp -crlf -connect <host>:25` to inspect certs and encrypted behavior.
- Use detected MTA name to choose targeted tests or known CVEs.

**Remediation**
- Minimize banner information and require auth where possible. Keep MTA patched.

---

## auxiliary/scanner/smtp/smtp_enum — User enumeration (VRFY/EXPN/RCPT)

**Purpose (short)**  
Probe for valid mailbox or system accounts using VRFY, EXPN, or RCPT verification methods.

**Important options**
- `RHOSTS`, `USER_FILE`, `THREADS`, `METHOD` (if module supports selecting VRFY/EXPN/RCPT).

**What it does**
- Iterates usernames and uses one or more verification methods to determine whether accounts exist.

**Typical output**
```
[+] 192.253.225.3:25 - Users found: admin, postmaster, www-data, backup
```

**Interpretation**
- Found usernames should be validated. System accounts often present. Use results to build credential lists.

**Caveats**
- Modern MTAs often disable VRFY/EXPN. RCPT TO checks may still reveal existence via different responses.
- False positives and rate-limiting possible.

**Follow-ups**
- Re-validate via manual `VRFY` or `RCPT TO` in a controlled session. Cross-check with other sources (LDAP, web userlists).

**Remediation**
- Disable VRFY/EXPN. Configure uniform responses to prevent account disclosure. Rate-limit RCPT attempts.

---

## auxiliary/scanner/smtp/smtp_relay — Open relay testing

**Purpose (short)**  
Test whether the MTA will relay mail from arbitrary senders to external recipients.

**Important options**
- `RHOSTS`, `FROM` and `TO` test lists, `THREADS`.

**What it does**
- Attempts to submit mail relay sequences and detects whether the server accepts relaying to external domains.

**Typical output**
```
[!] Server accepted relay for <attacker@evil.com> -> <victim@external.com> (OPEN RELAY)
```

**Interpretation**
- Open relay confirmed. High impact. Immediate remediation and notification required.

**Caveats**
- Some MTAs accept then reject later. Verify end-to-end if permitted.

**Follow-ups**
- Report and disable open relay. Check mailserver configuration (relay restrictions, TLS, auth requirements).

**Remediation**
- Restrict relaying to authenticated users and trusted networks. Apply connection controls and RBLs.

---

## auxiliary/scanner/smtp/smtp_ntlm_domain — NTLM / Domain discovery via SMTP

**Purpose (short)**  
Attempt to gather NTLM/domain info from MTAs that expose Windows auth/NTLM features or negotiate NTLM responses.

**Important options**
- `RHOSTS`, `THREADS`.

**What it does**
- Initiates auth sequences or inspects EHLO/advertised capabilities to find domain controller names, domain names, or NTLM data.

**Typical output**
```
[+] 192.253.225.3:25 - Domain: EXISTSE.LOCAL, Domain Controller: dc1.existse.local
```

**Interpretation**
- Reveals AD domain and DC names. Useful to correlate with SMB/LDAP enumeration.

**Caveats**
- Not all MTAs expose NTLM info. Some may require auth attempts to trigger responses.

**Follow-ups**
- Use SMB and LDAP enumeration to verify and expand domain context.

**Remediation**
- Avoid exposing domain controller hostnames in publicly visible services and require secured auth.

---

## auxiliary/scanner/smtp/smtp_vrfy — Focused VRFY checks

**Purpose (short)**  
Run targeted VRFY checks where supported. Some Metasploit builds separate this module.

**What it does**
- Sends VRFY commands for specific usernames and records responses.

**Typical manual VRFY example**
```bash
telnet demo.ine.local 25
EHLO attacker.xyz
VRFY admin
```
**Response example**
```
252 2.0.0 <admin> exists
```

**Interpretation**
- Indicates the account exists.

**Follow-ups**
- Add validated usernames to credential lists for cautious auth testing.

**Remediation**
- Disable VRFY and standardize responses to prevent user enumeration.

---

## Manual checks and supporting tools

### telnet / openssl s_client
- Manual interaction helps validate automated results.
```bash
telnet demo.ine.local 25
# or for STARTTLS
openssl s_client -starttls smtp -crlf -connect demo.ine.local:25
```

### smtp-user-enum
- External tool specialized for SMTP user enumeration. Use for cross-checking.

---

## INE lab example (complete scenario)

### Input
```bash
# Banner
use auxiliary/scanner/smtp/smtp_version
set RHOSTS demo.ine.local
run

# User enumeration
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS demo.ine.local
set USER_FILE /usr/share/wordlists/common_users.txt
run

# Relay test
use auxiliary/scanner/smtp/smtp_relay
set RHOSTS demo.ine.local
run

# NTLM/domain
use auxiliary/scanner/smtp/smtp_ntlm_domain
set RHOSTS demo.ine.local
run
```

### Output (aggregated)
```
[+] Banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
[+] Users found: _apt, admin, postmaster, www-data
[!] Server accepted relay for <attacker@evil.com> -> <victim@external.com> (OPEN RELAY)
[+] Domain: EXISTSE.LOCAL, DC: dc1.existse.local
```

### Short analysis and next steps
- MTA: Postfix identified. STARTTLS should be checked and certificates inspected.
- Users found should be validated and added to credential lists for authorized tests.
- Open relay is critical. Stop and report immediately.
- Domain discovery gives direction to SMB/LDAP enumeration. Correlate hostnames and test internal services.

---

## Final checklist (report)
- [ ] Save msfconsole raw output and loot files.
- [ ] Record RHOSTS, module options, and timestamps.
- [ ] Validate discovered users using manual VRFY/RCPT tests.
- [ ] If open relay found, notify owner and provide reproduction steps for patching.
- [ ] Correlate domain info with SMB/LDAP scans for follow-up.
