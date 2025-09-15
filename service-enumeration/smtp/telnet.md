# 📦 Telnet - SMTP Enumeration

This file explains how to use **Telnet** to manually interact with the SMTP service (port 25) and enumerate its features.

🔗 **For CLI tools like sendemail/swaks/msmtp, see [cli-tools.md](./cli-tools.md)**
---

## 🎯 Purpose

Using Telnet with SMTP helps you:
- Verify the SMTP service is running
- Identify the mail server software (e.g., Postfix, Exim, Sendmail)
- Enumerate supported SMTP extensions (VRFY, STARTTLS, etc.)
- Check message size limits and encoding support
- Send manual test emails to check local delivery and queue behavior (for authorized testing only)

---

## 🛠️ Steps

### 1. Connect to the SMTP Server
```bash
telnet demo.ine.local 25
```
✅ Example Response:
```
220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

---

### 2. Say Hello (HELO)
```bash
HELO attacker.xyz
```
✅ Response:
```
250 openmailbox.xyz
```

---

### 3. Extended Hello (EHLO)
```bash
EHLO attacker.xyz
```
✅ Example Response:
```
250-openmailbox.xyz
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250 SMTPUTF8
```

---

## 🔎 Key Findings

- **Mail Server Software**: Postfix  
- **Message Size Limit**: 10 MB  
- **VRFY Enabled**: Can be used for user enumeration  
- **STARTTLS Supported**: Allows encrypted sessions  
- **UTF-8 Support**: Extended character set in emails  

**Notes:**
- `VRFY` indicates the server may respond to username verification requests (useful for enumeration).
- `STARTTLS` indicates the server supports upgrading the connection to TLS.
- `SIZE` gives the maximum allowed message size in bytes.

---

## ✉️ Sending an Email Manually (MAIL FROM / RCPT TO / DATA)

This example shows a full SMTP transaction that results in the server accepting and queuing a message.

1. **Connect** (same as above)
2. **HELO/EHLO** (same as above)

3. **MAIL FROM:** (set sender)
```text
mail from: admin@attacker.xyz
```
**Response:**
```
250 2.1.0 Ok
```

4. **RCPT TO:** (set recipient)
```text
rcpt to: root@openmailbox.xyz
```
**Response:**
```
250 2.1.5 Ok
```

5. **DATA:** (start message body)
```text
data
```
**Response:**
```
354 End data with <CR><LF>.<CR><LF>
```

6. **Send headers and body, finish with a single dot on its own line**
```text
Subject: Hi Root
Hello,
This is a fake mail sent using telnet command.
From,
Admin
.
```
**Response:**
```
250 2.0.0 Ok: queued as 22BE774CFF41
```

**Interpretation:**
- `250 2.0.0 Ok: queued as <ID>` → message accepted and placed in the mail queue; `<ID>` is the queue ID.
- Accepting `MAIL FROM`/`RCPT TO` for local recipients without auth is common but should be carefully controlled to avoid abuse.

---

## 🔍 Things to Test / Follow-up Checks

- **VRFY / EXPN checks:** try `VRFY username` or `EXPN listname` to enumerate users/lists (if enabled).
- **RCPT TO external** (only on authorized tests) to check for open relay behavior.
- **STARTTLS validation:** use `openssl s_client -starttls smtp -connect demo.ine.local:25` to test TLS negotiation.
- **Check queue on the mail server** (if you have access): `postqueue -p` / `postcat -q <queue-id>` / `tail -n 200 /var/log/mail.log | grep <queue-id>`.

---

## ⚠️ Security & Rules of Engagement

- Only perform sending or relay tests against systems you own or have explicit permission to test.
- Manual SMTP interactions are logged on the server and can trigger alerts or rate-limiting.
- Avoid sending emails to real external recipients during tests.

---

## 🔁 Suggested Next Steps

- Cross-validate findings with `smtp-user-enum`, `smtp_enum` (Metasploit), and `RCPT TO` tests.
- Add any validated usernames to your password-guessing lists (authorized testing only).
- Document queue IDs and timestamps when reporting issues to server owners.
