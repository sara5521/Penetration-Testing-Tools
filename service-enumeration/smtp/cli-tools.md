# 🛠️ SMTP CLI Tools - sendemail / swaks / msmtp

This file collects examples of command-line tools (CLI) that can be used to send emails via SMTP directly from the shell (not inside a `telnet` session).
Use this file as a reference for commands, installation instructions, and security notes.

---

## 🔎 Important Note
**The commands here are executed in the shell (terminal) on your machine**, not inside a `telnet` session.  
If you want to perform the same dialogue manually via Telnet, see `service-enumeration/smtp/telnet.md`.

---

## 1) sendemail (Example)

### Installation (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install sendemail -y
```
> `sendemail` may not be available in all distributions. If not, use `swaks` or `msmtp` as alternatives.

### Basic usage (without TLS)
```bash
sendemail -f admin@attacker.xyz -t root@openmailbox.xyz -s demo.ine.local -u "Fakemail" -m "Hi root, a fake from admin" -o tls=no
```

### Usage with TLS (recommended when available)
```bash
sendemail -f admin@attacker.xyz -t root@openmailbox.xyz -s demo.ine.local -u "Fakemail" -m "Hi root" -o tls=yes
```

### Arguments explained
- `-f` : envelope sender (MAIL FROM)  
- `-t` : recipient (RCPT TO)  
- `-s` : SMTP server (host[:port])  
- `-u` : subject  
- `-m` : message body  
- `-o tls=no` : disable STARTTLS (testing only — not recommended on public networks)

---

## 2) swaks (Powerful alternative for testing)

### Installation
```bash
sudo apt update
sudo apt install swaks -y
```

### Simple example
```bash
swaks --to root@openmailbox.xyz --from admin@attacker.xyz --server demo.ine.local --header "Subject: Fakemail" --body "Hi root, test"
```

### Example with STARTTLS and authentication
```bash
swaks --to root@openmailbox.xyz --from admin@attacker.xyz --server demo.ine.local:25 --auth LOGIN --auth-user user --auth-password pass --tls
```

`swaks` is excellent for testing because it shows the full SMTP dialogue and gives detailed control over headers and auth methods.

---

## 3) msmtp (Useful for apps/scripts)

### Installation
```bash
sudo apt update
sudo apt install msmtp -y
```

### Simple usage
```bash
msmtp --host=demo.ine.local --from=admin@attacker.xyz root@openmailbox.xyz
# The message body is read from STDIN or from a file
```

`msmtp` is well-suited for use in scripts or applications to send mail programmatically.

---

## 🔍 Security Notes & Tips
- **Do not disable TLS on public networks** unless it is an authorized, controlled test.  
- Mail logs (`syslog`/`mail.log`) will record the sending activity, even if you use `sendemail`.  
- Avoid sending actual emails to external recipients during tests; use local/test inboxes.  
- When testing relay capabilities, never test against external domains without explicit permission.

---

## ✅ Related Documentation
- Manual Telnet sessions: `service-enumeration/smtp/telnet.md`  
- Automated enumeration tools: `service-enumeration/smtp/smtp-user-enum.md`  
- Metasploit SMTP modules: `service-enumeration/smtp/metasploit.md`

---
