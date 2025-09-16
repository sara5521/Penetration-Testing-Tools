# SSH — Nmap

## Purpose
Nmap commands and NSE scripts for SSH enumeration.

## Where to save
`service-enumeration/ssh/nmap.md`

## Useful commands
- Version detection:
```bash
nmap -sV -p 22 <target>
```
- SSH scripts:
```bash
nmap -p 22 --script ssh-hostkey,ssh2-enum-algos -sV <target>
```

## What results mean
- `ssh-hostkey` returns public host key fingerprint.

## Practical tips
- Save hostkey fingerprints for reporting.

## INE lab example
### Input
```bash
nmap -p 22 --script ssh-hostkey demo.ine.local
```
### Output
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1
| ssh-hostkey:
|   2048 xx:xx:xx:...
```
### Short analysis
- OpenSSH 7.6 detected and hostkey reported.
