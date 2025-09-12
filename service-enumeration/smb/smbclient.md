# 📂 smbclient

## ✅ Purpose
Check if anonymous (null session) access is allowed on the SMB server.

---

## 🛠️ Command Used in the Lab
```bash
smbclient -L demo.ine.local -N
```

### 🔍 Breakdown:
- `-L`: List shares
- `-N`: No password (anonymous login)

---

## 📸 Sample Output
```
Sharename       Type      Comment
---------       ----      -------
public          Disk      
john            Disk      
aisha           Disk      
emma            Disk      
everyone        Disk      
IPC$            IPC       IPC Service (samba.recon.lab)

Reconnecting with SMB1 for workgroup listing.

Server              Comment
---------           -----------------------------
Workgroup           Master
RECONLABS           SAMBA-RECON
```

---

## 🔐 Anonymous Access Allowed?
✅ Yes — Shares were listed without providing any credentials.

---

## 🎯 Why This Matters
- Anonymous access means you might:
  - View shared files without authentication
  - Download data from shared folders
  - Upload malicious files (if write access is allowed)

---

## 🔧 Related Tools
- `nmap --script smb-enum-shares`
- `enum4linux`
- `smbmap`
