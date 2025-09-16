# MySQL — Nmap

## Purpose
Nmap commands and NSE scripts for MySQL enumeration.

## Where to save
`service-enumeration/mysql/nmap.md`

## Useful commands
- Basic version:
```bash
nmap -sV -p 3306 <target>
```
- MySQL NSE scripts:
```bash
nmap -sV -p 3306 --script=mysql-info,mysql-empty-password <target>
```

## What results mean
- `mysql-info` returns version and plugin info.

## Practical tips
- Use mysql scripts with caution.

## INE lab example
### Input
```bash
nmap -sV -p 3306 --script=mysql-info demo.ine.local
```
### Output
```
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.7.33
| mysql-info: Version: 5.7.33
```
### Short analysis
- MySQL 5.7.33 detected. Next: test for empty passwords.
