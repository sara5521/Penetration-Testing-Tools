
# 🧭 nmblookup - SMB NetBIOS Name Lookup

The `nmblookup` tool helps you discover **NetBIOS names** of SMB servers. It uses **NetBIOS Name Service (NBNS)** over UDP port 137.

---

## 🎯 Purpose

- Discover the **NetBIOS hostname** of a Windows/Samba machine
- Identify workgroup/domain names
- Useful for mapping SMB hosts and verifying visibility

---

## 🧪 Basic Usage

```bash
nmblookup -A <target-ip or hostname>
```

**Example:**
```bash
nmblookup -A demo.ine.local
```

---

## 📸 Sample Output (INE Lab)

```bash
Looking up status of 192.220.69.3

    SAMBA-RECON    <00> - <ACTIVE>
    SAMBA-RECON    <03> - <ACTIVE>
    SAMBA-RECON    <20> - <ACTIVE>
    __MSBROWSE__   <01> - <GROUP> <ACTIVE>
    RECONLABS      <00> - <GROUP> <ACTIVE>
    RECONLABS      <1e> - <GROUP> <ACTIVE>
    MAC Address = 00-00-00-00-00-00
```

---

## 🧠 Interpretation

| Code | Description |
|------|-------------|
| `<00>` | Workstation service (NetBIOS name of the host) |
| `<03>` | Messenger service |
| `<20>` | File sharing service (Server Service) |
| `<01>` | Master browser (used in Windows Workgroup network) |
| `<1e>` | Browser service election |
| `<GROUP>` | Indicates group name (e.g. workgroup or domain) |

- ✅ `SAMBA-RECON` is the **NetBIOS computer name**
- 🏢 `RECONLABS` is the **workgroup/domain**
- These names are useful for SMB enumeration, domain targeting, or crafting usernames like `SAMBA-RECON\admin`.

---

## 📌 Notes

- `nmblookup` is part of the **Samba suite** (install with `apt install samba`).
- Similar to `nbtscan` but more focused.
- Works well in local networks or when NetBIOS is enabled.

---

## 🔁 Related Tools

- `nbtscan` → scans multiple IPs for NetBIOS names
- `nmap --script smb-os-discovery` → also shows NetBIOS name
- `enum4linux` → uses `nmblookup` internally

---

📚 [Official Samba Docs – nmblookup](https://www.samba.org/samba/docs/current/man-html/nmblookup.1.html)
