# 🛠️ rpcclient - Enumerate SMB Info via Null Session

The `rpcclient` tool can be used to enumerate information from an SMB server using a null session (i.e., without credentials).

---

## 🎯 Purpose

- Check if anonymous SMB connections are allowed
- Interact with Windows RPC (Remote Procedure Call) services
- Query users, groups, shares, and more (once connected)

---

## 🧪 Test Anonymous SMB Access

### 🔸 Command Used:
```bash
rpcclient -U "" -N demo.ine.local
```

| Option      | Meaning                                      |
|-------------|----------------------------------------------|
| `-U ""`     | Empty username (null session)                |
| `-N`        | No password required                         |
| `demo.ine.local` | Target hostname                        |

---

## ✅ Result from INE Lab:

```bash
rpcclient -U "" -N demo.ine.local
rpcclient $>
```

🟢 This prompt (`rpcclient $>`) confirms that:
- Anonymous connection is allowed.
- No credentials were required.
- You can now run enumeration commands inside the `rpcclient` shell.

---

## 🔍 What Can You Do Inside rpcclient?

Once connected, you can run commands like:
```bash
enumdomusers        # List all domain users
enumdomgroups       # List groups
querydominfo        # Get domain info
srvinfo             # System info (hostname, OS version, etc.)
netshareenumall     # List all shares
```

---

## 🧠 Why Is This Useful?

- Gain information without credentials (great for stealth).
- Can be used for:
  - User enumeration
  - Share enumeration
  - Domain info gathering
- Often enabled by misconfigured Windows or Samba servers.

---

## 🔐 Related Tools

| Tool        | Purpose                                  |
|-------------|------------------------------------------|
| `smbclient` | Check for accessible shares              |
| `nmap` NSE  | Run SMB-related scripts                  |
| `enum4linux`| Wrapper for smbclient & rpcclient tools  |

