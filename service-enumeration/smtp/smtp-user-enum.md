# 📦 smtp-user-enum - SMTP User Enumeration

The `smtp-user-enum` tool automates enumeration of SMTP users by sending commands like **VRFY**, **EXPN**, or **RCPT TO**.

---

## 🎯 Purpose

- Identify valid usernames on a mail server
- Automate what can be done manually with Telnet
- Useful for identifying accounts like `root`, `admin`, or service accounts

---

## 🛠️ Usage

```bash
smtp-user-enum -U <user-list.txt> -t <target-ip>
```

### Example (INE Lab)
```bash
smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t demo.ine.local
```

✅ Example Output:
```
existse.local: admin
existse.local: administrator
existse.local: mail
existse.local: postmaster
existse.local: root
existse.local: sales
existse.local: support
demo.ine.local: www-data exists
```

---

## 🔎 Key Findings

- Valid users were discovered:
  - `admin`
  - `administrator`
  - `mail`
  - `postmaster`
  - `root`
  - `sales`
  - `support`
  - `www-data`
- This information can be used later for **brute force attacks** or privilege escalation.

---

## 💡 Tips

- Modes available:
  - `-M VRFY` → Verify users (default).
  - `-M EXPN` → Expand mailing lists.
  - `-M RCPT` → Test recipient delivery.
- Always use a **small wordlist first** to avoid triggering defenses.
