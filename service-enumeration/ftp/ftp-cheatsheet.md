# FTP Cheat-sheet — quick reference
Path: service-enumeration/ftp/ftp-cheatsheet.md

## Quick msfconsole commands
msfconsole -q
use auxiliary/scanner/ftp/ftp_version
set RHOSTS <target>
run

use auxiliary/scanner/ftp/ftp_login
set RHOSTS <target>
set USER_FILE /path/users.txt
set PASS_FILE /path/pass.txt
set STOP_ON_SUCCESS true
run

use auxiliary/scanner/ftp/anonymous
set RHOSTS <target>
run

## CLI quick commands
ftp <target>
Name: <user>
password: <pass>
ftp> ls
ftp> get <file>
ftp> put <file>   # only in lab

## Useful nmap scripts
nmap -p21 --script ftp-syst,ftp-anon <target>
nmap -p21 --script ftp-vsftpd-backdoor <target>

## Evidence tips
- Save msf console output with: spool ~/logs/ftp_<target>_msf.log
- Save nmap xml with -oA <name> for reproducibility
- Store evidence in service-enumeration/ftp/evidence/
