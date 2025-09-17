# 🎯 Metasploit - HTTP Enumeration Modules

This section covers Metasploit auxiliary modules for comprehensive HTTP/web application enumeration and security testing.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Module Overview](#-module-overview)
- [Prerequisites](#-prerequisites)
- [HTTP Version Detection](#-1-http-version-detection)
- [Content Discovery](#-2-content-discovery)
- [Directory Enumeration](#-3-directory-enumeration)
- [File Upload Testing](#-4-file-upload-testing)
- [Authentication Testing](#-5-authentication-testing)
- [Advanced Techniques](#-6-advanced-techniques)
- [Output Analysis](#-7-output-analysis-and-interpretation)
- [Integration with Other Tools](#-8-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

Metasploit HTTP modules help you:
- Detect web server versions and technologies
- Discover hidden directories and sensitive files
- Test file upload and deletion capabilities
- Perform authentication brute-force attacks
- Enumerate user directories and configurations
- Identify security misconfigurations
- Gather reconnaissance data for web application testing

---

## 🗂️ Module Overview

| Module | Purpose | Risk Level | Auth Required |
|--------|---------|------------|---------------|
| `http_version` | Server version detection | 🟢 Low | ❌ |
| `robots_txt` | Robots.txt enumeration | 🟢 Low | ❌ |
| `http_header` | Header analysis | 🟢 Low | ❌ |
| `brute_dirs` | Quick directory brute-force | 🟡 Medium | ❌ |
| `dir_scanner` | Wordlist-based directory scan | 🟡 Medium | ❌ |
| `dir_listing` | Directory listing check | 🟡 Medium | ❌ |
| `files_dir` | Sensitive file discovery | 🟡 Medium | ❌ |
| `http_put` | File upload/delete testing | 🔴 High | ❌ |
| `http_login` | Authentication brute-force | 🔴 High | ❌ |
| `apache_userdir_enum` | User directory enumeration | 🟡 Medium | ❌ |

---

## 📋 Prerequisites

### Environment Setup:
```bash
# Start Metasploit
msfconsole -q

# Update Metasploit (if needed)
msfupdate

# Check available HTTP modules
search type:auxiliary http
```

### Target Requirements:
- HTTP/HTTPS service running (port 80/443)
- Network connectivity to target
- Proper authorization for testing

### Quick Service Check:
```bash
# From Metasploit console
use auxiliary/scanner/portscan/tcp
set RHOSTS <target-ip>
set PORTS 80,443
run
```

---

## 🔧 Metasploit HTTP Modules

## 🏷️ 1. HTTP Version Detection

### Module: `auxiliary/scanner/http/http_version`

**📌 Purpose:**
Detect web server software and version information for fingerprinting and vulnerability research.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/http_version
set RHOSTS <target-ip>
run
```

**⚙️ Module Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `RHOSTS` | Target host(s) | - | `192.168.1.100` |
| `RPORT` | Target port | `80` | `8080` |
| `SSL` | Enable HTTPS | `false` | `true` |
| `THREADS` | Scan threads | `1` | `10` |
| `VHOST` | Virtual host | - | `example.com` |

**📌 INE Lab Example:**
```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
set SSL false
run
```

**📸 Sample Output:**
```bash
[*] 192.74.12.3:80 Apache/2.4.18 (Ubuntu)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### Advanced Configuration:
```bash
# HTTPS scanning
use auxiliary/scanner/http/http_version
set RHOSTS <target>
set RPORT 443
set SSL true
run

# Multiple hosts
set RHOSTS 192.168.1.0/24
set THREADS 20
run
```

---

## 📝 2. Content Discovery

### Module: `auxiliary/scanner/http/robots_txt`

**📌 Purpose:**
Retrieve and analyze robots.txt files to discover hidden directories and sensitive paths.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS <target-ip>
run
```

**📸 Sample Output:**
```bash
[*] Checking robots.txt
[+] Contents of Robots.txt:
User-agent: *
Disallow: /admin
Disallow: /backup
Disallow: /config
Disallow: /data
Allow: /public
```

### Module: `auxiliary/scanner/http/http_header`

**📌 Purpose:**
Collect and analyze HTTP headers for security configurations and technology detection.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/http_header
set RHOSTS <target-ip>
set TARGETURI /
run
```

**⚙️ Key Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `TARGETURI` | Specific path to test | `/` | `/admin` |
| `METHOD` | HTTP method | `GET` | `POST` |

**📸 Sample Output:**
```bash
[*] 192.168.1.100:80 :SERVER: Apache/2.4.18 (Ubuntu)
[*] 192.168.1.100:80 :X-POWERED-BY: PHP/7.4.3
[*] 192.168.1.100:80 :CONTENT-TYPE: text/html; charset=UTF-8
[*] 192.168.1.100:80 :WWW-AUTHENTICATE: Basic realm="Restricted Content"
```

---

## 📂 3. Directory Enumeration

### Module: `auxiliary/scanner/http/brute_dirs`

**📌 Purpose:**
Quick directory brute-force using built-in common directory list.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS <target-ip>
run
```

**📸 Sample Output:**
```bash
[+] Found http://192.168.1.100:80/admin/
[+] Found http://192.168.1.100:80/login/
[+] Found http://192.168.1.100:80/backup/
```

### Module: `auxiliary/scanner/http/dir_scanner`

**📌 Purpose:**
Advanced directory scanning using custom wordlists for comprehensive enumeration.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS <target-ip>
set DICTIONARY /usr/share/wordlists/dirb/common.txt
run
```

**⚙️ Advanced Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `DICTIONARY` | Wordlist path | - | `/path/to/wordlist.txt` |
| `THREADS` | Parallel threads | `1` | `10` |
| `PATH` | Base path | `/` | `/app/` |

**📸 Sample Output:**
```bash
[+] 192.168.1.100:80/admin/ exists (200 OK)
[+] 192.168.1.100:80/config/ exists (403 Forbidden)
[+] 192.168.1.100:80/backup/ exists (200 OK)
[+] 192.168.1.100:80/uploads/ exists (200 OK)
```

### Module: `auxiliary/scanner/http/dir_listing`

**📌 Purpose:**
Check for enabled directory listing and enumerate exposed files.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS <target-ip>
set PATH /uploads
run
```

**📸 Sample Output:**
```bash
[+] Directory listing is enabled at /uploads/
[*] Found the following files:
    - document.pdf
    - backup.zip
    - config.txt
    - image.jpg
```

---

## 🔍 4. Sensitive File Discovery

### Module: `auxiliary/scanner/http/files_dir`

**📌 Purpose:**
Search for common sensitive files like backups, configuration files, and logs.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/files_dir
set RHOSTS <target-ip>
run
```

**📸 Sample Output:**
```bash
[+] Found http://192.168.1.100:80/config.php.bak
[+] Found http://192.168.1.100:80/.env
[+] Found http://192.168.1.100:80/backup.sql
[+] Found http://192.168.1.100:80/error.log
```

### Custom File Discovery:
```bash
#!/bin/bash
# Custom sensitive file checker with Metasploit

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Sensitive File Discovery ==="

# Common sensitive files
sensitive_files=(
    ".env"
    "config.php.bak"
    "backup.sql"
    "database.sql"
    ".git/config"
    ".htaccess"
    "web.config"
    "phpinfo.php"
    "test.php"
    "robots.txt"
    "sitemap.xml"
)

cat > /tmp/sensitive_files.rc << EOF
$(for file in "${sensitive_files[@]}"; do
    echo "use auxiliary/scanner/http/files_dir"
    echo "set RHOSTS $TARGET"
    echo "set FILES $file"
    echo "run"
done)
exit
EOF

msfconsole -q -r /tmp/sensitive_files.rc
rm -f /tmp/sensitive_files.rc
```

---

## 📤 5. File Upload Testing

### Module: `auxiliary/scanner/http/http_put`

**📌 Purpose:**
Test HTTP PUT method for file upload and deletion capabilities.

**⚠️ WARNING:** This module can modify server content. Use only with proper authorization.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/http_put
set RHOSTS <target-ip>
set PATH /uploads
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```

**⚙️ Module Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `PATH` | Upload directory | `/` | `/uploads/` |
| `FILENAME` | File name | `msf_http_put_test.txt` | `test.txt` |
| `FILEDATA` | File content | `msf_http_put_test` | `<script>alert('xss')</script>` |
| `ACTION` | Action type | `PUT` | `DELETE` |

**📸 Sample Output:**
```bash
[+] Successfully uploaded file: /uploads/test.txt
[*] File should be available at: http://192.168.1.100:80/uploads/test.txt
```

### File Upload Validation:
```bash
# After upload, verify file accessibility
use auxiliary/scanner/http/http_header
set RHOSTS <target>
set TARGETURI /uploads/test.txt
run

# Or use external verification
!wget http://<target>/uploads/test.txt -O downloaded_file.txt
!cat downloaded_file.txt
```

### File Deletion Testing:
```bash
use auxiliary/scanner/http/http_put
set RHOSTS <target>
set PATH /uploads
set FILENAME test.txt
set ACTION DELETE
run
```

---

## 🔐 6. Authentication Testing

### Module: `auxiliary/scanner/http/http_login`

**📌 Purpose:**
Brute-force web login forms and HTTP Basic authentication.

**⚠️ WARNING:** This can lock accounts and trigger security alerts. Use with extreme caution.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/http_login
set RHOSTS <target-ip>
set AUTH_URI /login
set USERNAME admin
set PASSWORD password123
run
```

**⚙️ Advanced Options:**
| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `AUTH_URI` | Login endpoint | `/` | `/admin/login` |
| `USERNAME` | Single username | - | `admin` |
| `PASSWORD` | Single password | - | `password123` |
| `USER_FILE` | Username wordlist | - | `/path/to/users.txt` |
| `PASS_FILE` | Password wordlist | - | `/path/to/passwords.txt` |
| `STOP_ON_SUCCESS` | Stop after first success | `false` | `true` |

**📝 Wordlist-Based Attack:**
```bash
use auxiliary/scanner/http/http_login
set RHOSTS <target>
set AUTH_URI /admin/login
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
set THREADS 5
run
```

**📸 Sample Output:**
```bash
[*] Attempting to login to http://192.168.1.100:80/admin/login
[-] 192.168.1.100:80 - Login failed: 'admin:admin'
[-] 192.168.1.100:80 - Login failed: 'admin:password'
[+] 192.168.1.100:80 - Login successful: 'admin:password123'
```

### Module: `auxiliary/scanner/http/apache_userdir_enum`

**📌 Purpose:**
Enumerate Apache user directories (~user) to discover personal web spaces.

**🔧 Basic Usage:**
```bash
use auxiliary/scanner/http/apache_userdir_enum
set RHOSTS <target-ip>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
run
```

**📸 Sample Output:**
```bash
[+] Found userdir: http://192.168.1.100/~alice/
[+] Found userdir: http://192.168.1.100/~bob/
[+] Found userdir: http://192.168.1.100/~admin/
```

---

## 🚀 7. Advanced Techniques

### Automated HTTP Assessment:
```bash
#!/bin/bash
# Comprehensive HTTP assessment using Metasploit

TARGET=$1
OUTPUT_DIR="http_assessment_$(date +%Y%m%d_%H%M%S)"

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"
echo "=== HTTP Assessment Pipeline ==="
echo "Target: $TARGET"
echo "Output: $OUTPUT_DIR"

# Create comprehensive assessment resource file
cat > "$OUTPUT_DIR/http_assessment.rc" << EOF
# HTTP Assessment Resource File
spool $OUTPUT_DIR/metasploit_output.txt

# Step 1: Version detection
use auxiliary/scanner/http/http_version
set RHOSTS $TARGET
run

# Step 2: Content discovery
use auxiliary/scanner/http/robots_txt
set RHOSTS $TARGET
run

use auxiliary/scanner/http/http_header
set RHOSTS $TARGET
run

# Step 3: Directory enumeration
use auxiliary/scanner/http/brute_dirs
set RHOSTS $TARGET
run

use auxiliary/scanner/http/dir_scanner
set RHOSTS $TARGET
set DICTIONARY /usr/share/wordlists/dirb/common.txt
set THREADS 10
run

# Step 4: File discovery
use auxiliary/scanner/http/files_dir
set RHOSTS $TARGET
run

# Step 5: User directory enumeration
use auxiliary/scanner/http/apache_userdir_enum
set RHOSTS $TARGET
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
run

spool off
exit
EOF

# Run assessment
echo "Running Metasploit assessment..."
msfconsole -q -r "$OUTPUT_DIR/http_assessment.rc"

# Generate summary report
echo "Generating summary..."
cat > "$OUTPUT_DIR/summary.txt" << EOF
HTTP Assessment Summary
Target: $TARGET
Date: $(date)

== Server Information ==
$(grep -E "\[.*\].*Apache|nginx|IIS|Microsoft" "$OUTPUT_DIR/metasploit_output.txt" | head -5)

== Discovered Directories ==
$(grep -E "Found http://|exists \(200" "$OUTPUT_DIR/metasploit_output.txt" | head -10)

== Sensitive Files ==
$(grep -E "Found.*\.(bak|sql|env|config)" "$OUTPUT_DIR/metasploit_output.txt")

== User Directories ==
$(grep "Found userdir:" "$OUTPUT_DIR/metasploit_output.txt")
EOF

echo "Assessment complete. Results in $OUTPUT_DIR/"
cat "$OUTPUT_DIR/summary.txt"
```

### Database Integration:
```bash
# Enable workspace
workspace -a http_assessment

# Run modules (results auto-saved)
use auxiliary/scanner/http/http_version
set RHOSTS 192.168.1.0/24
run

# Query results
hosts
services -p 80,443
notes
```

### Custom Wordlist Creation:
```bash
#!/bin/bash
# Create custom wordlist for target

TARGET_DOMAIN=$1
if [[ -z "$TARGET_DOMAIN" ]]; then
    echo "Usage: $0 <domain>"
    exit 1
fi

echo "Creating custom wordlist for $TARGET_DOMAIN"

# Common directories
common_dirs=(
    "admin" "login" "dashboard" "panel" "control"
    "backup" "config" "data" "files" "uploads"
    "api" "v1" "v2" "test" "dev" "staging"
    "wp-admin" "wp-content" "wp-includes"
    "phpmyadmin" "phpinfo" "info"
)

# Technology-specific paths
tech_paths=(
    "app" "application" "src" "assets" "static"
    "js" "css" "images" "media" "resources"
    "includes" "lib" "libraries" "vendor"
    "cache" "temp" "tmp" "logs" "log"
)

# Create combined wordlist
{
    printf '%s\n' "${common_dirs[@]}"
    printf '%s\n' "${tech_paths[@]}"
    
    # Add domain-specific variations
    echo "${TARGET_DOMAIN%.*}-admin"
    echo "${TARGET_DOMAIN%.*}-backup"
    echo "${TARGET_DOMAIN%.*}-dev"
    
} > custom_wordlist.txt

echo "Custom wordlist created: custom_wordlist.txt ($(wc -l < custom_wordlist.txt) entries)"
```

---

## 🔍 8. Output Analysis and Interpretation

### Server Version Analysis:

**Apache Server:**
```bash
[*] 192.168.1.100:80 Apache/2.4.18 (Ubuntu)
```
- **Software**: Apache HTTP Server
- **Version**: 2.4.18 - check for CVEs
- **OS**: Ubuntu - Linux environment
- **Action**: Research Apache 2.4.18 vulnerabilities

**Nginx Server:**
```bash
[*] 192.168.1.100:80 nginx/1.18.0
```
- **Software**: Nginx web server
- **Version**: 1.18.0 - relatively recent
- **Action**: Check for Nginx-specific misconfigurations

### Directory Enumeration Analysis:

**Response Code Interpretation:**
- **200 OK**: Directory/file exists and is accessible
- **301/302**: Redirect - may indicate real directory
- **401**: Authentication required
- **403**: Forbidden - exists but access denied
- **404**: Not found

**High-Value Findings:**
```bash
[+] Found http://target/admin/ (200)
[+] Found http://target/config/ (403)  
[+] Found http://target/backup/ (200)
[+] Found http://target/.env (200)
```
- **admin/**: Administrative interface
- **config/**: Configuration files (forbidden but exists)
- **backup/**: Backup files - high priority
- **.env**: Environment file - critical finding

### Authentication Results:

**Successful Login:**
```bash
[+] 192.168.1.100:80 - Login successful: 'admin:password123'
```
- **Impact**: Unauthorized access to web application
- **Next Steps**: Document session cookies, test privilege levels
- **Evidence**: Save request/response pairs

### File Upload Results:

**Successful Upload:**
```bash
[+] Successfully uploaded file: /uploads/test.txt
```
- **Risk Level**: High - potential for web shell upload
- **Validation**: Verify file is accessible and executable
- **Cleanup**: Delete test files after assessment

---

## 🔗 9. Integration with Other Tools

### With Nmap:
```bash
# Initial discovery
nmap -p 80,443 --script http-enum <target>

# Follow up with Metasploit
use auxiliary/scanner/http/http_version
set RHOSTS <target>
run
```

### With Burp Suite:
```bash
# Use Metasploit findings to configure Burp
# 1. Import discovered directories into Burp's site map
# 2. Configure authentication with discovered credentials
# 3. Test upload functionality with Burp's file upload scanner
```

### With Gobuster:
```bash
# Cross-validate directory findings
gobuster dir -u http://<target>/ -w /usr/share/wordlists/dirb/common.txt

# Use Metasploit for specific file testing
use auxiliary/scanner/http/files_dir
set RHOSTS <target>
run
```

### Pipeline Integration:
```bash
#!/bin/bash
# HTTP enumeration to exploitation pipeline

TARGET=$1
OUTPUT_DIR="http_pipeline"
mkdir -p "$OUTPUT_DIR"

echo "=== HTTP Enumeration to Exploitation Pipeline ==="

# Phase 1: Reconnaissance
echo "Phase 1: Reconnaissance"
nmap -sV -p 80,443 --script http-enum "$TARGET" > "$OUTPUT_DIR/nmap_http.txt"

# Phase 2: Metasploit enumeration
echo "Phase 2: Metasploit enumeration"
cat > "$OUTPUT_DIR/msf_recon.rc" << EOF
use auxiliary/scanner/http/http_version
set RHOSTS $TARGET
run
use auxiliary/scanner/http/robots_txt
set RHOSTS $TARGET
run
use auxiliary/scanner/http/dir_scanner
set RHOSTS $TARGET
set DICTIONARY /usr/share/wordlists/dirb/common.txt
run
exit
EOF

msfconsole -q -r "$OUTPUT_DIR/msf_recon.rc" > "$OUTPUT_DIR/msf_recon.txt"

# Phase 3: Analyze findings
echo "Phase 3: Analyzing findings"
if grep -q "Found.*admin" "$OUTPUT_DIR/msf_recon.txt"; then
    echo "Admin interface found - testing authentication"
    
    cat > "$OUTPUT_DIR/msf_auth.rc" << EOF
use auxiliary/scanner/http/http_login
set RHOSTS $TARGET
set AUTH_URI /admin/login
set USERNAME admin
set PASSWORD password123
run
exit
EOF
    
    msfconsole -q -r "$OUTPUT_DIR/msf_auth.rc" > "$OUTPUT_DIR/msf_auth.txt"
fi

# Phase 4: File upload testing (if authorized)
if [[ "$2" == "upload-test" ]]; then
    echo "Phase 4: File upload testing"
    cat > "$OUTPUT_DIR/msf_upload.rc" << EOF
use auxiliary/scanner/http/http_put
set RHOSTS $TARGET
set PATH /uploads
set FILENAME test.txt
set FILEDATA "Test upload"
run
exit
EOF
    
    msfconsole -q -r "$OUTPUT_DIR/msf_upload.rc" > "$OUTPUT_DIR/msf_upload.txt"
fi

echo "Pipeline complete. Results in $OUTPUT_DIR/"
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Error Message | Solution |
|-------|---------------|----------|
| **Module Not Found** | `Failed to load module` | Update Metasploit framework |
| **Connection Timeout** | `Connection timed out` | Check network connectivity |
| **SSL Handshake Failed** | `SSL connect error` | Set SSL option correctly |
| **Authentication Failed** | `Login failed` | Verify credentials and login endpoint |
| **No Results Found** | Empty output | Check target responsiveness and paths |

### Debugging Commands:
```bash
# Check Metasploit version
version

# Update framework
msfupdate

# Test basic connectivity
use auxiliary/scanner/portscan/tcp
set RHOSTS <target>
set PORTS 80,443
run

# Enable verbose output
set VERBOSE true
```

### Performance Optimization:
```bash
# Increase threads for faster scanning
set THREADS 20

# Set shorter timeouts
set ConnectTimeout 5
set SendTimeout 5

# Use targeted wordlists
set DICTIONARY /usr/share/wordlists/dirb/small.txt
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Written Authorization**: Always obtain proper authorization before testing
- **Scope Compliance**: Only test systems explicitly included in scope
- **Data Protection**: Handle discovered data responsibly
- **File Upload Risks**: HTTP PUT testing can modify server content

### Operational Security:
- **Logging Impact**: HTTP enumeration generates extensive web server logs
- **WAF Detection**: Web Application Firewalls may block or rate-limit requests
- **Account Lockouts**: Authentication testing can lock user accounts
- **Service Disruption**: Aggressive scanning may impact server performance

### Detection Indicators:
- **Access Logs**: Unusual request patterns and user agents
- **Error Logs**: Multiple 404 errors from directory brute-forcing
- **Security Alerts**: Failed authentication attempts
- **File System Changes**: Uploaded test files

### Defensive Countermeasures:
- **Web Application Firewall**: Deploy WAF to filter malicious requests
- **Rate Limiting**: Implement request rate limiting per IP
- **Authentication Controls**: Use strong passwords and account lockout policies
- **File Upload Security**: Restrict upload capabilities and file types
- **Monitoring**: Deploy comprehensive web application monitoring

### Risk Assessment:
| Activity | Detection Risk | Impact | Recommendation |
|----------|----------------|--------|----------------|
| Version Detection | 🟢 Low | 🟢 Low | Generally safe for reconnaissance |
| Directory Enumeration | 🟠 High | 🟡 Medium | May trigger WAF alerts |
| Authentication Testing | 🔴 High | 🔴 High | Can lock accounts, use carefully |
| File Upload Testing | 🔴 High | 🔴 High | Only with explicit authorization |

---

## 📖 References

### Official Documentation:
- [Metasploit Framework](https://docs.rapid7.com/metasploit/) - Official documentation
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) - Free training course
- [Rapid7 Blog](https://blog.rapid7.com/) - Latest research and techniques

### Security Research:
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Web application security risks
- [Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) - Comprehensive testing methodology
- [HTTP Security Headers](https://owasp.org/www-project-secure-headers/) - Security header reference

### Learning Resources:
- [Web Application Hacker's Handbook](https://www.wiley.com/en-us/The+Web+Application+Hacker%27s+Handbook-p-9781118026472) - Comprehensive web security
- [Burp Suite Documentation](https://portswigger.net/burp/documentation) - Professional web testing
- [HTTP Enumeration Techniques](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web) - Practical methodology

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Version detection
use auxiliary/scanner/http/http_version; set RHOSTS <target>; run

# Content discovery
use auxiliary/scanner/http/robots_txt; set RHOSTS <target>; run

# Directory enumeration  
use auxiliary/scanner/http/dir_scanner; set RHOSTS <target>; set DICTIONARY /path/to/wordlist.txt; run

# File upload testing
use auxiliary/scanner/http/http_put; set RHOSTS <target>; set PATH /uploads; set FILENAME test.txt; run
```

### Module Priority:
1. **High Priority**: `http_version`, `robots_txt` (safe reconnaissance)
2. **Medium Priority**: `dir_scanner`, `files_dir` (enumeration)
3. **Use with Caution**: `http_login`, `http_put` (active testing)

### Key Information to Collect:
- Web server software and version
- Discovered directories and files
- Authentication mechanisms
- File upload capabilities
- Security header configurations

### Follow-up Tools:
1. Burp Suite for detailed web application testing
2. Gobuster/Dirbuster for comprehensive directory brute-forcing
3. Nikto for web server vulnerability scanning
4. Custom scripts for targeted testing

### Success Indicators:
- Server version identified
- Hidden directories discovered
- Sensitive files found
- Authentication bypass achieved
- File upload capability confirmed

### Time Estimates:
- **Version detection**: 30 seconds per host
- **Basic enumeration**: 5-15 minutes per host
- **Directory brute-force**: 10-60 minutes (depending on wordlist)
- **Authentication testing**: 10+ minutes (depending on wordlist)
- **Comprehensive assessment**: 1-3 hours per target

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting HTTP security assessments.*
