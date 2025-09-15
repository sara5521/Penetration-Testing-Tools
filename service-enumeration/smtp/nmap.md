# 📦 Nmap - SMTP Enumeration Scripts

This section includes useful Nmap scripts to enumerate **SMTP (Simple Mail Transfer Protocol)** services.

---

## 🎯 Purpose

These scripts help you gather important information from the SMTP service, such as:
- Service banner (hostname, mail server software, greeting message)
- Supported SMTP commands
- Valid user accounts (if server allows enumeration)
- Authentication methods
- Security settings

---

## 🗂️ SMTP Enumeration Scripts Summary

| # | Script             | What it Does                           |
|---|--------------------|----------------------------------------|
| 1 | banner             | Grab the SMTP banner (server greeting) |

---

## 1️⃣ Grab SMTP Banner

```bash
nmap -sV -p 25 --script banner <target-ip>
```

**INE Lab Example**
```bash
nmap -sV -p 25 --script banner demo.ine.local
```

**📸 Sample Output:**
```bash
25/tcp open  smtp  Postfix smtpd
|_banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

**Interpretation:**
- 📨 Service: Postfix SMTP daemon
- 👋 Banner: "Welcome to our mail server"
- 🌍 Hostname: `openmailbox.xyz`
- ⚠️ Banners can be faked — use them only as hints

---

📖 [View Nmap SMTP Scripts](https://nmap.org/nsedoc/categories/smtp.html)
