# SSH — CLI Tools

## Purpose
Basic SSH commands and quick checks.

## Where to save
`service-enumeration/ssh/cli-tools.md`

## Key commands
- Check SSH banner:
```bash
nc <target> 22
# or
ssh -v <user>@<target>
```
- Test connecting with known keys:
```bash
ssh -i id_rsa user@<target>
```

## How to read output
- `SSH-2.0-OpenSSH_7.6p1` — server and version.

## Practical tips
- Use `ssh -v` for verbose debug.

## INE lab example
### Input
```bash
nc demo.ine.local 22
```
### Output
```
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```
### Short analysis
- OpenSSH 7.6 detected. Use nmap scripts to enumerate algorithms.
