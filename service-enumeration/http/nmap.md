# HTTP — Nmap

## Purpose
Nmap commands and HTTP-related NSE scripts for enumeration.

## Where to save
`service-enumeration/http/nmap.md`

## Useful commands
- Version detection:
```bash
nmap -sV -p 80,443 <target>
```
- HTTP scripts, robots and headers:
```bash
nmap -p 80 --script http-robots.txt,http-title,http-enum -sV <target>
```

## What results mean
- `http-robots.txt` finds disallowed paths.
- `http-title` shows site title.

## Practical tips
- Combine nmap and gobuster for deeper discovery.

## INE lab example
### Input
```bash
nmap -p 80 --script http-robots.txt,http-title demo.ine.local
```
### Output
```
| http-robots.txt:  Disallow: /secret
| http-title: Example Site
```
### Short analysis
- robots disallows /secret. Use directory brute-force and check that path.
