# FTP — Nmap

## Purpose
Nmap commands and NSE scripts for FTP enumeration.

## Where to save
`service-enumeration/ftp/nmap.md`

## Useful commands
- Basic service/version:
```bash
nmap -sV -p 21 <target>
```
- Banner and FTP scripts:
```bash
nmap -p 21 --script ftp-anon,ftp-brute,ftp-syst -sV <target>
```

## What results mean
- `ftp-anon` shows anonymous login allowed.
- `ftp-syst` shows server system type.

## Practical tips
- Run `-sV` first then targeted scripts.

## INE lab example
### Input
```bash
nmap -sV -p 21 --script ftp-syst demo.ine.local
```
### Output
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: SYST: UNIX Type: L8
```
### Short analysis
- vsftpd detected. System type is UNIX. Next: try anonymous login.
