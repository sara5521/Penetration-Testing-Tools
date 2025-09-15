# 📦 Telnet - SMTP Enumeration

This file explains how to use **Telnet** to manually interact with the SMTP service (port 25) and enumerate its features.

---

## 🎯 Purpose

Using Telnet with SMTP helps you:
- Verify the SMTP service is running
- Identify the mail server software (e.g., Postfix, Exim, Sendmail)
- Enumerate supported SMTP extensions (VRFY, STARTTLS, etc.)
- Check message size limits and encoding support

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

---
