# MySQL — CLI Tools

## Purpose
Basic commands to interact with MySQL services from CLI.

## Where to save
`service-enumeration/mysql/cli-tools.md`

## Key commands
- Check banner with netcat:
```bash
nc <target> 3306
```
- Use mysql client (if credentials known):
```bash
mysql -h <target> -u <user> -p
```

## How to read output
- MySQL greeting line shows version.

## Practical tips
- Use `nmap --script mysql-info` before manual login attempts.

## INE lab example
### Input
```bash
nc demo.ine.local 3306
```
### Output
```
5.7.33-0ubuntu0.18.04.1
```
### Short analysis
- MySQL version revealed. Use mysql-info NSE next.
