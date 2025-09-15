# 📦 Netcat - SMTP Enumeration

This section explains how to use **Netcat** to manually interact with the SMTP service and enumerate users with the `VRFY` command.

---

## 🎯 Purpose

Using Netcat with SMTP helps you:
- Connect directly to the mail server on port 25.
- Read the SMTP banner (hostname, software, greeting).
- Manually test commands like `VRFY` and `EXPN`.
- Discover valid usernames based on server responses.
- Understand common SMTP reply codes.

---

## 🗂️ SMTP Netcat Commands Summary

| # | Command / Step                 | What it Does                                |
|---|--------------------------------|---------------------------------------------|
| 1 | `nc <host> 25`                 | Connect to the SMTP service                 |
| 2 | `VRFY <user or email>`         | Ask if a user/email exists                  |
| 3 | `QUIT`                         | Close the SMTP connection cleanly           |

**Common SMTP reply codes:**
- **220** → Server ready (banner/greeting)
- **250** → OK (user exists)
- **252** → Cannot VRFY but will accept mail (maybe valid)
- **550** → User not found / rejected

---

## 1️⃣ Connect & Read Banner

**Input**
```bash
nc demo.ine.local 25
```

**Output**
```
220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

**Interpretation**
- **220** = Service ready.  
- Host greets as **openmailbox.xyz** (Postfix server).  
- Banner shows the mail server type and welcome message.  

---

## 2️⃣ Verify an Existing User

**Input**
```bash
VRFY admin@openmailbox.xyz
```

**Output**
```
252 2.0.0 admin@openmailbox.xyz
```

**Interpretation**
- **252** = The server will accept mail for this address but does not confirm if the user exists.  
- Treated as **possibly valid**.  
- Some servers always return 252 to prevent easy enumeration.  

---

## 3️⃣ Verify a Non-Existing User

**Input**
```bash
VRFY commander@openmailbox.xyz
```

**Output**
```
550 5.1.1 <commander@openmailbox.xyz>: Recipient address rejected: User unknown in local recipient table
```

**Interpretation**
- **550** = User does not exist.  
- This error is a strong signal that the tested account is invalid.  

---

## 🧪 Quick Workflow

```bash
# 1) Connect to SMTP
nc demo.ine.local 25

# 2) Try verifying users
VRFY admin
VRFY admin@openmailbox.xyz
VRFY commander@openmailbox.xyz

# 3) Quit the session
QUIT
```

**What to record in your notes/report:**
- Banner details (hostname, mail software).
- Each command tested and its exact response.
- Mark accounts as **Valid** (250), **Possibly valid** (252), or **Invalid** (550).

---

## 🔎 Extra Tips

- If `VRFY` and `EXPN` are disabled, try:  
  - `RCPT TO:<user@domain>` in a fake mail session to check validity.  
- Always cross-check with automated scripts/tools like:  
  ```bash
  nmap -p 25 --script=smtp-commands,smtp-enum-users demo.ine.local
  ```
- Combine with dedicated tools like `smtp-user-enum` for larger userlists.  
- Only use this on **authorized targets** (labs, testing environments).

---
