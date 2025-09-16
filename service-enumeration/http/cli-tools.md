# HTTP — CLI Tools

## Purpose
Basic CLI commands for interacting with web servers.

## Where to save
`service-enumeration/http/cli-tools.md`

## Key commands
- Fetch headers:
```bash
curl -I http://<target>/
```
- View full response:
```bash
curl -v http://<target>/
```
- TLS check:
```bash
openssl s_client -connect <target>:443 -servername <host>
```

## How to read output
- `HTTP/1.1 200 OK` — page returned.
- `Server:` header reveals web server.

## Practical tips
- Use `curl -I` for headers only.

## INE lab example
### Input
```bash
curl -I http://demo.ine.local/
```
### Output
```
HTTP/1.1 200 OK
Server: Apache/2.4.41 (Ubuntu)
```
### Short analysis
- Server is Apache. Next: run directory enumeration.
