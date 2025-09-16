# FTP — CLI Tools

## Purpose
Quick commands to interact with FTP services using common CLI tools.

## Where to save
`service-enumeration/ftp/cli-tools.md`

## Key commands
- List FTP server banner:
```bash
telnet <target> 21
# or
nc <target> 21
```
- Anonymous login and list files:
```bash
ftp <target>
# then: user anonymous ""
#       ls
```
- Using `curl` to check responses:
```bash
curl -v ftp://<target>/
```

## How to read output
- `220` banner — server ready.
- `230` — user logged in.
- `530` — authentication required/failed.

## Practical tips
- Try anonymous login first.
- Use `nc` for quick banner grab.

## INE lab example
### Input
```bash
nc demo.ine.local 21
```
### Output
```
220 (vsFTPd 3.0.3)
```
### Short analysis
- FTP server is running vsFTPd. Try anonymous login and list files.
