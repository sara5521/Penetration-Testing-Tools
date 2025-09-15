# 📦 Netcat – SMTP Enumeration (VRFY)

This section shows how to use **netcat** to talk to an SMTP server and check if users exist using `VRFY`.

---

## 📂 Path in GitHub Project
```
service-enumeration/smtp/netcat.md
```

---

## 🎯 Purpose
With netcat you can:
- Connect to the SMTP service (port **25**).
- Read the **banner** (server name / greeting).
- Send **VRFY** to test if an email/user exists.
- Understand common **SMTP reply codes** (e.g., `220`, `250/252`, `550`).

---

## 🗂️ Commands Summary

| # | Command / Step                  | What it Does                                  |
|---|---------------------------------|-----------------------------------------------|
| 1 | `nc <host> 25`                  | Connect to SMTP and read the banner           |
| 2 | `VRFY <user or email>`          | Ask server to verify a user/email             |
| 3 | `QUIT`                          | Close the session cleanly                      |

**Common reply codes you’ll see:**
- **220** → Server ready (banner/greeting)
- **250** → OK (user exists) *(if server reveals)*
- **252** → “Cannot VRFY, but will accept message” → **maybe valid**, not confirmed
- **550** → User unknown / not found

---

## 🖥️ 1) Connect & Read Banner

**Input**
```bash
nc demo.ine.local 25
```

**Output**
```
220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

**Interpretation (simple):**
- **220** = SMTP server is ready.
- Host greets as **openmailbox.xyz** (Postfix).
- Banners are **hints** (can be customized/spoofed).

---

## 👤 2) Verify a Known/Existing User (VRFY)

**Input**
```bash
VRFY admin@openmailbox.xyz
```

**Output**
```
252 2.0.0 admin@openmailbox.xyz
```

**Interpretation (simple):**
- **252** means the server **won’t confirm** if the user exists, but it will **accept mail and try delivery**.
- Treat `252` as **“possibly valid”** (not guaranteed).  

> Tip: Try both formats: `VRFY admin` and `VRFY admin@openmailbox.xyz`.

---

## ❌ 3) Verify a Non-Existing User (VRFY)

**Input**
```bash
VRFY commander@openmailbox.xyz
```

**Output**
```
550 5.1.1 <commander@openmailbox.xyz>: Recipient address rejected: User unknown in local recipient table
```

**Interpretation (simple):**
- **550** = user **does not exist** on this server.
- Clear negative signal → good for building a **valid/invalid** user list.

---

## 🧪 Quick Workflow

```bash
# 1) Connect
nc demo.ine.local 25
# Server shows: 220 ...

# 2) Try verify a few users
VRFY admin
VRFY admin@openmailbox.xyz
VRFY commander@openmailbox.xyz

# 3) Quit
QUIT
```

**What to log in your notes/report:**
- Banner text (hostname/software)
- Each `VRFY` tested and the **exact** reply code/message
- Mark users as: **Valid** (250), **Possibly valid** (252), or **Invalid** (550)

---

## 🔎 Extra Tips
- If `VRFY` is blocked or always returns `252`, try:
  - `EXPN` (if enabled) – expands lists/aliases
  - `RCPT TO:<user@domain>` during a fake mail flow to infer validity  
- Cross-check with Nmap:
```bash
nmap -p 25 --script=smtp-commands,smtp-enum-users demo.ine.local
```
- Always do this **with permission** (labs/authorized targets only).

