# 📧 Email Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate email services (SMTP, POP3, IMAP) on mail servers and gather intelligence for security assessments.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-email-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [SMTP Banner Grabbing](#-1-grab-smtp-banner)
- [SMTP Service Discovery](#-2-smtp-service-information-and-capabilities)
- [SMTP User Enumeration](#-3-smtp-user-enumeration)
- [SMTP Security Testing](#-4-smtp-security-and-relay-testing)
- [POP3 Service Enumeration](#-5-pop3-service-enumeration)
- [IMAP Service Enumeration](#-6-imap-service-enumeration)
- [SSL/TLS Email Security](#-7-ssltls-email-security)
- [Authentication Testing](#-8-email-authentication-testing)
- [Complete Assessment Workflow](#-complete-email-service-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from email services, such as:
- Service banner (hostname, mail server software, greeting message)
- Supported SMTP commands and capabilities
- Valid user accounts (if server allows enumeration)
- Authentication methods and security settings
- Open relay configuration
- SSL/TLS encryption support
- POP3 and IMAP service capabilities
- Mail server vulnerabilities

---

## 🗂️ Email Enumeration Scripts Summary

| # | Script                 | What it Does                             | Auth Required | Protocols |
|---|------------------------|------------------------------------------|---------------|-----------|
| 1 | banner                 | Grab service banner/greeting             | ❌            | SMTP/POP3/IMAP |
| 2 | smtp-commands          | Show supported SMTP commands             | ❌            | SMTP |
| 3 | smtp-capabilities      | Show SMTP capabilities and features      | ❌            | SMTP |
| 4 | smtp-enum-users        | Enumerate valid email users             | ❌            | SMTP |
| 5 | smtp-open-relay        | Test for open mail relay                | ❌            | SMTP |
| 6 | smtp-brute             | Brute force SMTP authentication         | ❌            | SMTP |
| 7 | pop3-capabilities      | Show POP3 capabilities                   | ❌            | POP3 |
| 8 | pop3-brute             | Brute force POP3 authentication         | ❌            | POP3 |
| 9 | imap-capabilities      | Show IMAP capabilities                   | ❌            | IMAP |
| 10 | imap-brute            | Brute force IMAP authentication         | ❌            | IMAP |
| 11 | imap-ntlm-info        | Extract NTLM info from IMAP              | ❌            | IMAP |
| 12 | ssl-enum-ciphers      | Test SSL/TLS encryption strength         | ❌            | Secure Email |

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has email services enabled** (ports 25, 110, 143, 465, 587, 993, 995)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick Email Port Check:
```bash
# Check all standard email ports
nmap -sV -p 25,110,143,465,587,993,995 <target-ip>

# Check specifically for SMTP
nmap -p 25,465,587 <target-ip>
```

### Standard Email Ports:
- **25**: SMTP (unencrypted)
- **110**: POP3 (unencrypted)
- **143**: IMAP (unencrypted)
- **465**: SMTPS (SMTP over SSL)
- **587**: SMTP with STARTTLS
- **993**: IMAPS (IMAP over SSL)
- **995**: POP3S (POP3 over SSL)

---

## 🔧 Email Service Enumeration Commands

## 📨 1. Grab SMTP Banner
```bash
nmap -sV -p 25,465,587 --script banner <target-ip>
```
**INE Lab Example:**
```bash
nmap -sV -p 25 --script banner demo.ine.local
```
**Sample Output:**
```bash
25/tcp open  smtp  Postfix smtpd
|_banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```
**Interpretation:**
- Service: Postfix SMTP daemon
- Banner: "Welcome to our mail server"
- Hostname: `openmailbox.xyz`
- Protocol: ESMTP (Extended SMTP)

**Pro Tip:** Banners can be customized or faked, but they often reveal valuable information about the mail server software and version.

---

## 🌐 2. SMTP Service Information and Capabilities
```bash
# SMTP service capabilities
nmap --script smtp-capabilities -p 25,465,587 <target-ip>

# SMTP supported commands
nmap --script smtp-commands -p 25,465,587 <target-ip>

# Combined SMTP information gathering
nmap --script smtp-commands,smtp-capabilities -p 25,465,587 <target-ip>
```
**Purpose:**

These scripts discover what features and commands the SMTP server supports, such as:
- Authentication methods (PLAIN, LOGIN, CRAM-MD5)
- Size limits for messages
- STARTTLS support for encryption
- Extended SMTP features

**INE Lab Example:**
```bash
nmap --script smtp-commands -p 25 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smtp-commands: 
|   EHLO demo.ine.local
|   250-PIPELINING
|   250-SIZE 10240000
|   250-VRFY
|   250-ETRN
|   250-ENHANCEDSTATUSCODES
|   250-8BITMIME
|   250-DSN
|   250-SMTPUTF8
|_  250 CHUNKING
```
**Interpretation:**
- Server supports pipelining for faster mail processing
- Maximum message size: 10,240,000 bytes (~10MB)
- VRFY command enabled (potential for user enumeration)
- Enhanced status codes supported
- UTF8 email addresses supported

---

## 👥 3. SMTP User Enumeration
```bash
# Basic SMTP user enumeration
nmap --script smtp-enum-users -p 25 <target-ip>

# SMTP VRFY command user enumeration
nmap --script smtp-enum-users --script-args smtp-enum-users.methods=VRFY -p 25 <target-ip>

# SMTP EXPN command user enumeration
nmap --script smtp-enum-users --script-args smtp-enum-users.methods=EXPN -p 25 <target-ip>

# SMTP RCPT TO user enumeration
nmap --script smtp-enum-users --script-args smtp-enum-users.methods=RCPT -p 25 <target-ip>

# Combined SMTP user enumeration methods
nmap --script smtp-enum-users --script-args smtp-enum-users.methods={VRFY,EXPN,RCPT} -p 25 <target-ip>
```
**Purpose:**

This script attempts to enumerate valid email addresses/usernames on the SMTP server using different methods:
- **VRFY**: Verify if a user exists
- **EXPN**: Expand mailing lists
- **RCPT**: Test recipient addresses

**INE Lab Example:**
```bash
nmap --script smtp-enum-users --script-args smtp-enum-users.methods=VRFY -p 25 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smtp-enum-users: 
|   VRFY:
|     admin@demo.ine.local: Valid user
|     john@demo.ine.local: Valid user
|     mary@demo.ine.local: Valid user
|     test@demo.ine.local: Valid user
|_    root@demo.ine.local: Valid user
```
**Interpretation:**
- The server responds to VRFY commands (not all servers do)
- Several valid email addresses discovered
- These addresses can be used for targeted phishing or password attacks
- Administrative accounts (admin, root) identified

**Attack Implications:**
- Valid email addresses for social engineering attacks
- Username format pattern discovered
- Administrative accounts for privilege escalation attempts

---

## 🔐 4. SMTP Security and Relay Testing
```bash
# SMTP open relay testing
nmap --script smtp-open-relay -p 25 <target-ip>

# SMTP vulnerability assessment
nmap --script smtp-vuln-* -p 25,465,587 <target-ip>

# SMTP security assessment combined
nmap --script smtp-open-relay,smtp-capabilities,smtp-commands -p 25 <target-ip>
```
**Purpose:**

Open relay testing determines if the SMTP server can be used to send unauthorized emails (spam/phishing).

**INE Lab Example:**
```bash
nmap --script smtp-open-relay -p 25 demo.ine.local
```
**Sample Output:**
```bash
Host script results:
| smtp-open-relay: 
|_  Server is not an open relay.
```
or
```bash
Host script results:
| smtp-open-relay: 
|   SMTP open relay detected!
|   Relaying from: 192.168.1.100
|   To: external@example.com
|_  This server can be used to send spam!
```

**Security Implications:**
- **Not Open Relay**: Server properly configured, only allows authorized sending
- **Open Relay**: Server vulnerable to abuse for spam/phishing campaigns
- Open relays are often blacklisted by major email providers

---

## 📬 5. POP3 Service Enumeration
```bash
# Basic POP3 assessment
nmap --script pop3-capabilities -p 110,995 <target-ip>

# POP3 brute force authentication
nmap --script pop3-brute -p 110,995 <target-ip>

# POP3 brute force with custom credentials
nmap --script pop3-brute --script-args userdb=users.txt,passdb=passwords.txt -p 110,995 <target-ip>

# All POP3 scripts
nmap --script pop3-* -p 110,995 <target-ip>
```
**Purpose:**

POP3 enumeration reveals capabilities and tests authentication on Post Office Protocol services.

**Sample Output:**
```bash
Host script results:
| pop3-capabilities: 
|   CAPA:
|     TOP
|     RESP-CODES
|     AUTH-RESP-CODE
|     PIPELINING
|     USER
|_    UIDL
```
**Interpretation:**
- Server supports TOP command (retrieve message headers)
- UIDL supported for unique message identification
- No SASL authentication methods listed (may use basic auth only)

---

## 📨 6. IMAP Service Enumeration
```bash
# Basic IMAP assessment
nmap --script imap-capabilities -p 143,993 <target-ip>

# IMAP brute force authentication
nmap --script imap-brute -p 143,993 <target-ip>

# IMAP brute force with custom wordlists
nmap --script imap-brute --script-args userdb=users.txt,passdb=passwords.txt -p 143,993 <target-ip>

# IMAP NTLM information disclosure
nmap --script imap-ntlm-info -p 143,993 <target-ip>

# All IMAP scripts
nmap --script imap-* -p 143,993 <target-ip>
```
**Purpose:**

IMAP enumeration discovers Internet Message Access Protocol capabilities and configuration.

**Sample Output:**
```bash
Host script results:
| imap-capabilities: 
|   CAPABILITY:
|     IMAP4REV1
|     LITERAL+
|     SASL-IR
|     LOGIN-REFERRALS
|     ID
|     ENABLE
|     IDLE
|     AUTH=PLAIN
|_    AUTH=LOGIN
```
**Interpretation:**
- IMAP version 4 revision 1 supported
- Authentication methods: PLAIN and LOGIN
- IDLE command supported (push notifications)
- Server supports SASL authentication framework

---

## 🔒 7. SSL/TLS Email Security
```bash
# SSL/TLS cipher assessment for secure email ports
nmap --script ssl-enum-ciphers -p 465,587,993,995 <target-ip>

# SSL certificate information
nmap --script ssl-cert -p 465,587,993,995 <target-ip>

# Combined SSL/TLS assessment
nmap --script ssl-enum-ciphers,ssl-cert -p 465,587,993,995 <target-ip>

# Check for SSL/TLS support on standard ports
nmap --script ssl-enum-ciphers -p 25,110,143 <target-ip>
```
**Purpose:**

Assess the security of encrypted email connections and identify weak encryption.

**Sample Output:**
```bash
| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 - A
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA - A
|     compressors: 
|       NULL
|   TLSv1.3: 
|     ciphers: 
|       TLS_AES_256_GCM_SHA384 - A
|       TLS_CHACHA20_POLY1305_SHA256 - A
|_    TLS_AES_128_GCM_SHA256 - A
```

---

## 🔑 8. Email Authentication Testing
```bash
# SMTP authentication brute force
nmap --script smtp-brute -p 25,465,587 <target-ip>

# SMTP brute force with custom wordlists
nmap --script smtp-brute --script-args userdb=emails.txt,passdb=passwords.txt -p 25,587 <target-ip>

# POP3 authentication testing
nmap --script pop3-brute --script-args brute.firstonly=true -p 110,995 <target-ip>

# IMAP authentication testing
nmap --script imap-brute --script-args brute.threads=3,brute.delay=1 -p 143,993 <target-ip>

# Combined email authentication testing
nmap --script smtp-brute,pop3-brute,imap-brute --script-args brute.firstonly=true -p 25,110,143 <target-ip>
```
**Purpose:**

Test for weak or default credentials on email services.

**Security Considerations:**
- Use `brute.firstonly=true` to stop after first successful login
- Use `brute.delay` to avoid account lockouts
- Limit `brute.threads` to avoid overwhelming the service

---

## 🎯 Complete Email Service Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: Email port discovery
nmap -sS -p 25,110,143,465,587,993,995 <target>

# Step 2: Service version detection
nmap -sV -p 25,110,143,465,587,993,995 <target>

# Step 3: Basic banner information
nmap --script banner -p 25,110,143,465,587,993,995 <target>
```

### Phase 2: Service Capabilities and Configuration
```bash
# Step 4: SMTP capabilities and commands
nmap --script smtp-commands,smtp-capabilities -p 25,465,587 <target>

# Step 5: POP3 and IMAP capabilities
nmap --script pop3-capabilities,imap-capabilities -p 110,143,993,995 <target>

# Step 6: SSL/TLS configuration assessment
nmap --script ssl-enum-ciphers,ssl-cert -p 465,587,993,995 <target>
```

### Phase 3: Security Testing and User Enumeration
```bash
# Step 7: SMTP open relay testing
nmap --script smtp-open-relay -p 25 <target>

# Step 8: User enumeration via SMTP
nmap --script smtp-enum-users -p 25 <target>

# Step 9: Authentication testing (if appropriate)
nmap --script smtp-brute,pop3-brute,imap-brute --script-args brute.firstonly=true -p 25,110,143 <target>
```

---

## 🔁 All-in-One Scan

### Basic Email Discovery (no authentication)
```bash
nmap -sV -p 25,110,143,465,587,993,995 --script "smtp-commands,smtp-capabilities,pop3-capabilities,imap-capabilities,banner" <target-ip>
```

### Security Assessment Scan
```bash
nmap -p 25,110,143,465,587,993,995 --script "smtp-open-relay,smtp-enum-users,ssl-enum-ciphers" <target-ip>
```

### Comprehensive Email Assessment
```bash
nmap --script smtp-*,pop3-*,imap-*,ssl-enum-ciphers,ssl-cert -p 25,110,143,465,587,993,995 <target-ip>
```

### Authentication Testing (Use Carefully)
```bash
nmap --script smtp-brute,pop3-brute,imap-brute --script-args brute.firstonly=true,brute.delay=2 -p 25,110,143 <target-ip>
```

---

## 📊 Real Lab Examples

### Example 1: SMTP Server Assessment with User Enumeration
```bash
# Target: SMTP server with potential user enumeration

# Service detection
nmap -sV -p 25,465,587 192.168.1.100

# SMTP capabilities
nmap --script smtp-commands -p 25 192.168.1.100

# User enumeration via VRFY
nmap --script smtp-enum-users --script-args smtp-enum-users.methods=VRFY -p 25 192.168.1.100

# Open relay testing
nmap --script smtp-open-relay -p 25 192.168.1.100
```

**Expected Output:**
```
PORT    STATE SERVICE VERSION
25/tcp  open  smtp    Postfix smtpd
465/tcp open  smtps   Postfix smtpd
587/tcp open  smtp    Postfix smtpd

| smtp-commands: 
|_  EHLO, STARTTLS, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN

| smtp-enum-users: 
|   VRFY:
|     admin@company.com: Valid user
|     john@company.com: Valid user
|     mary@company.com: Valid user
|_    test@company.com: Valid user

| smtp-open-relay: 
|_  Server is not an open relay.
```

### Example 2: Microsoft Exchange Assessment
```bash
# Target: Exchange server detection and enumeration

# Multi-protocol service detection
nmap -sV -p 25,110,143,465,587,993,995,80,443 192.168.1.200

# Exchange-specific enumeration
nmap --script http-title,smtp-*,imap-* -p 25,80,443,587,993 192.168.1.200

# SSL/TLS assessment for Exchange
nmap --script ssl-enum-ciphers,ssl-cert -p 443,465,587,993,995 192.168.1.200
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Refused** | `Connection refused` | Check if email service is running |
| **Timeout Issues** | Scripts hang or timeout | Use `--script-args timeout=30s` |
| **No User Enumeration** | Empty VRFY results | Server may have VRFY disabled |
| **SSL Errors** | SSL handshake failures | Check supported SSL/TLS versions |
| **Authentication Failed** | Login failures | Verify credentials and account status |

### Common Error Messages:
- `501 VRFY command is disabled` → User enumeration not possible
- `530 Authentication required` → Server requires login before accepting commands
- `550 Relay not permitted` → Properly configured (not open relay)
- `Connection timed out` → Firewall or service not running

### Performance Optimization:
```bash
# Faster email scanning
nmap -T4 --script smtp-commands,smtp-enum-users -p 25 <target>

# Slower scanning to avoid detection
nmap -T2 --script smtp-* -p 25,465,587 <target>

# Parallel authentication testing with delays
nmap --script smtp-brute --script-args brute.threads=2,brute.delay=3 -p 25 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **Be cautious with brute force attacks** - they can cause account lockouts
- **Follow responsible disclosure** if vulnerabilities are found

### Operational Security (OpSec):
- **Email enumeration generates logs** - be aware of detection
- **Brute force attacks are noisy** - use sparingly and with delays
- **SMTP relay testing may trigger alerts** - monitor for defensive responses
- **SSL/TLS testing is generally safe** but may log unusual cipher requests

### Detection Indicators:
- **Email Logs**: Authentication attempts, VRFY command usage
- **Network Monitoring**: Unusual SMTP traffic patterns
- **Failed Login Attempts**: Multiple authentication failures
- **Relay Testing**: Attempts to send test emails

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| Open SMTP Relay | 🔴 Critical | Configure relay restrictions immediately |
| VRFY Command Enabled | 🟡 Medium | Disable VRFY to prevent user enumeration |
| Weak SSL/TLS Ciphers | 🟠 High | Update cipher suites and disable weak encryption |
| Default Credentials | 🔴 Critical | Change all default passwords |
| Unencrypted Email Ports | 🟠 High | Enforce SSL/TLS for all email communications |

---

## 🎯 eJPT Focus Points

### Essential Email Commands for eJPT:
```bash
# Email service discovery
nmap -sV -p 25,110,143,465,587,993,995 <target>

# SMTP user enumeration
nmap --script smtp-enum-users -p 25 <target>

# Open relay testing
nmap --script smtp-open-relay -p 25 <target>

# Email authentication brute force
nmap --script smtp-brute,pop3-brute,imap-brute --script-args brute.firstonly=true -p 25,110,143 <target>
```

### Key Email Attack Vectors for eJPT:
1. **User Enumeration**: SMTP VRFY/EXPN commands for username discovery
2. **Open Relay**: SMTP servers allowing unauthorized email relay
3. **Authentication Bypass**: Weak or default credentials on email services
4. **Information Disclosure**: Email server banners revealing software versions
5. **SSL/TLS Issues**: Weak encryption or missing secure connections
6. **Configuration Errors**: Misconfigured authentication mechanisms

### Critical Email Information for Further Testing:
- **Valid Email Addresses**: Targets for social engineering and password attacks
- **Email Server Software**: Version information for vulnerability research
- **Authentication Methods**: Supported mechanisms for credential attacks
- **SSL/TLS Configuration**: Encryption strength and certificate information
- **Domain Information**: Corporate domains for further reconnaissance

---

## 🔗 Integration with Other Tools

After Nmap email enumeration, follow up with:
- **smtp-user-enum**: Advanced SMTP user enumeration
- **swaks**: Swiss Army Knife for SMTP testing
- **telnet/netcat**: Manual email protocol interaction
- **hydra**: Advanced authentication brute forcing
- **Metasploit**: Email service exploitation modules
- **theHarvester**: Email address gathering from public sources
- **SpiderFoot**: Automated reconnaissance including email discovery

### Email Security Testing Workflow:
```bash
# 1. Nmap email discovery and enumeration
nmap --script smtp-*,pop3-*,imap-* -p 25,110,143,465,587,993,995 <target>

# 2. Extract discovered users for further testing
grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' email_results.txt > email_addresses.txt

# 3. Advanced SMTP user enumeration
smtp-user-enum -M VRFY -U usernames.txt -t <target>

# 4. Password attacks against discovered accounts  
hydra -L email_addresses.txt -P passwords.txt <target> smtp

# 5. Manual email server interaction and testing
telnet <target> 25
```

---

## 📝 Output and Documentation

### Saving Email Assessment Results:
```bash
# Comprehensive email assessment
nmap --script smtp-*,pop3-*,imap-* -oA email_enumeration -p 25,110,143,465,587,993,995 <target>

# SMTP user enumeration results
nmap --script smtp-enum-users -oN smtp_users.txt -p 25 <target>

# Email server security assessment
nmap --script smtp-open-relay,ssl-enum-ciphers -oX email_security.xml -p 25,465,587,993,995 <target>
```

### Results Processing and Analysis:
```bash
# Extract discovered email addresses
grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' email_enumeration.nmap

# Find open relay servers
grep -i "open relay\|relay.*open" email_enumeration.nmap

# Identify authentication mechanisms
grep -i "auth\|login\|plain\|md5" email_enumeration.nmap
```

---

## 📖 References and Further Reading

### Official Documentation:
- [Nmap NSE Scripts](https://nmap.org/nsedoc/scripts/) - Complete NSE script reference
- [SMTP Protocol RFC 5321](https://tools.ietf.org/html/rfc5321) - SMTP specification
- [POP3 Protocol RFC 1939](https://tools.ietf.org/html/rfc1939) - POP3 specification
- [IMAP Protocol RFC 3501](https://tools.ietf.org/html/rfc3501) - IMAP specification

### Security Research:
- [OWASP Email Security](https://owasp.org/www-community/vulnerabilities/Email_Security) - Email security testing guide
- [SANS Email Security](https://www.sans.org/white-papers/email-security/) - Professional email security practices

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Scope**: Complete Email Service Enumeration Guide
