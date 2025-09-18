# 🌐 HTTP/HTTPS Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate HTTP/HTTPS services and assess web application security for penetration testing and security assessments.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-http-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [Basic HTTP Discovery](#-1-basic-http-service-discovery)
- [Directory and Path Discovery](#-2-directory-and-path-discovery)
- [HTTP Methods Testing](#-3-http-methods-testing)
- [Web Application Security](#-4-web-application-security-testing)
- [WebDAV Enumeration](#-5-webdav-enumeration)
- [SSL/TLS Assessment](#-6-ssltls-and-https-testing)
- [CMS and Framework Detection](#-7-content-management-systems-and-frameworks)
- [Authentication Testing](#-8-authentication-testing)
- [Advanced Techniques](#-9-advanced-http-enumeration)
- [Complete Assessment Workflow](#-complete-http-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from HTTP/HTTPS services, such as:
- Web server software and version identification
- Directory and file discovery
- HTTP methods and capabilities
- WebDAV functionality and upload capabilities
- SSL/TLS configuration and security
- Content Management System detection
- Authentication mechanisms
- Web application vulnerabilities
- Custom headers and security configurations

---

## 🗂️ HTTP Enumeration Scripts Summary

| # | Script                    | What it Does                             | Auth Required | Ports |
|---|---------------------------|------------------------------------------|---------------|-------|
| 1 | http-enum                 | Directory and file discovery             | ❌            | 80,443,8080 |
| 2 | http-title                | Extract web page titles                  | ❌            | 80,443,8080 |
| 3 | http-server-header        | Extract server header information        | ❌            | 80,443,8080 |
| 4 | http-methods              | Test HTTP methods (GET, POST, PUT, etc.) | ❌            | 80,443,8080 |
| 5 | http-headers              | Analyze HTTP response headers            | ❌            | 80,443,8080 |
| 6 | http-webdav-scan          | WebDAV service detection                 | ❌            | 80,443,8080 |
| 7 | http-brute                | HTTP authentication brute force         | ❌            | 80,443,8080 |
| 8 | http-default-accounts     | Test for default credentials             | ❌            | 80,443,8080 |
| 9 | http-vuln-*               | Web vulnerability scanning               | ❌            | 80,443,8080 |
| 10 | ssl-cert                 | SSL certificate information              | ❌            | 443,8443 |
| 11 | ssl-enum-ciphers         | SSL/TLS cipher enumeration               | ❌            | 443,8443 |

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has HTTP/HTTPS services enabled** (ports 80, 443, 8080, etc.)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick HTTP Port Check:
```bash
# Check standard web ports
nmap -sV -p 80,443,8080,8443 <target-ip>

# Check extended web ports
nmap -p 80,443,8080,8443,8000,8888,9000,9090,10000 <target-ip>
```

### Check if HTTP services are responsive:
```bash
nmap --script http-title -p 80,443 <target-ip>
```

---

## 🔧 HTTP Service Enumeration Commands

## 🌐 1. Basic HTTP Service Discovery
```bash
# Basic HTTP service detection
nmap -sV -p 80,443,8080 <target-ip>

# HTTP title and server information
nmap --script http-title,http-server-header -p 80,443 <target-ip>

# Basic HTTP headers analysis
nmap --script http-headers -p 80,443 <target-ip>

# Comprehensive basic HTTP information
nmap -sV --script http-title,http-server-header,http-headers -p 80,443,8080 <target-ip>
```
**INE Lab Example:**
```bash
nmap -sV --script http-title,http-server-header -p 80 demo.ine.local
```
**Sample Output:**
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
**Interpretation:**
- Web server: Microsoft IIS version 10.0
- Default IIS page is served
- Windows Server environment
- No custom web application detected

**Why is this useful:**
- Server version reveals potential vulnerabilities
- Default pages indicate lack of hardening
- Technology stack identification for targeted attacks

---

## 📁 2. Directory and Path Discovery
```bash
# Standard directory enumeration
nmap --script http-enum -p 80,443 <target-ip>

# Display all findings including negatives
nmap --script http-enum --script-args http-enum.displayall -p 80,443 <target-ip>

# Custom base path enumeration
nmap --script http-enum --script-args http-enum.basepath=/admin/ -p 80 <target-ip>

# Multiple interesting paths
nmap --script http-enum --script-args http-enum.basepath=/uploads/ -p 80 <target-ip>
nmap --script http-enum --script-args http-enum.basepath=/webdav/ -p 80 <target-ip>
nmap --script http-enum --script-args http-enum.basepath=/backup/ -p 80 <target-ip>
```
**Purpose:**

Directory enumeration discovers hidden or interesting directories and files on web servers, such as:
- Administrative interfaces
- Upload directories
- Backup files
- Configuration files
- Database files

**INE Lab Example:**
```bash
nmap --script http-enum -sV -p 80 demo.ine.local
```
**Sample Output:**
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-enum: 
|   /webdav/: Potentially interesting folder (401 Unauthorized)
|   /admin/: Potentially interesting folder
|   /uploads/: Potentially interesting folder
|   /backup/: Potentially interesting folder
|_  /config/: Potentially interesting folder
|_http-server-header: Microsoft-IIS/10.0
```
**Interpretation:**
- **WebDAV directory found**: `/webdav/` requires authentication (401)
- **Administrative area**: `/admin/` may contain management interface
- **Upload directory**: `/uploads/` potential for file upload attacks
- **Backup directory**: `/backup/` may contain sensitive files
- **Configuration files**: `/config/` may expose system settings

**Attack Implications:**
- WebDAV may allow file uploads if credentials are found
- Admin directories are prime targets for privilege escalation
- Backup files often contain sensitive information

---

## 🔧 3. HTTP Methods Testing
```bash
# Test HTTP methods on root directory
nmap --script http-methods -p 80,443 <target-ip>

# Test methods on specific path
nmap --script http-methods --script-args http-methods.url-path=/webdav/ <target-ip>

# Test all methods including dangerous ones
nmap --script http-methods --script-args http-methods.test-all -p 80,443 <target-ip>

# Multiple path testing
nmap --script http-methods --script-args http-methods.url-path=/admin/ -p 80 <target-ip>
nmap --script http-methods --script-args http-methods.url-path=/uploads/ -p 80 <target-ip>
```
**Purpose:**

HTTP methods testing reveals what operations are allowed on different directories:
- **GET/POST**: Standard web browsing
- **PUT/DELETE**: File upload/deletion capabilities
- **PROPFIND/LOCK**: WebDAV functionality
- **TRACE**: Potentially dangerous debugging method

**INE Lab Example:**
```bash
nmap --script http-methods --script-args http-methods.url-path=/webdav/ demo.ine.local
```
**Sample Output:**
```bash
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE COPY PROPFIND LOCK UNLOCK
|_  Potentially risky methods: TRACE COPY PROPFIND LOCK UNLOCK
```
**Interpretation:**
- **Standard methods**: GET, HEAD, POST, OPTIONS (normal web functionality)
- **WebDAV methods**: COPY, PROPFIND, LOCK, UNLOCK (file management)
- **Dangerous method**: TRACE (can reveal request details in responses)

**Security Assessment:**
- WebDAV methods indicate file upload/download capabilities
- TRACE method can be used for Cross-Site Tracing (XST) attacks
- PUT/DELETE methods allow file manipulation if not properly secured

---

## 🛡️ 4. Web Application Security Testing
```bash
# Common web vulnerabilities scan
nmap --script http-vuln-* -p 80,443 <target-ip>

# SQL injection detection
nmap --script http-sql-injection -p 80,443 <target-ip>

# Cross-site scripting detection
nmap --script http-stored-xss,http-dombased-xss -p 80,443 <target-ip>

# Directory traversal testing
nmap --script http-passwd,http-traverse -p 80,443 <target-ip>

# Backup file detection
nmap --script http-backup-finder -p 80,443 <target-ip>

# Comprehensive vulnerability assessment
nmap --script http-sql-injection,http-stored-xss,http-passwd,http-backup-finder -p 80,443 <target-ip>
```
**Purpose:**

Web application security testing identifies common vulnerabilities that can lead to:
- Data breaches through SQL injection
- Client-side attacks via XSS
- Information disclosure through directory traversal
- Sensitive file exposure

**Sample Vulnerability Output:**
```bash
| http-sql-injection: 
|   Possible sqli for queries:
|     http://target/search.php?q=test%27
|_    http://target/login.php?username=admin%27

| http-backup-finder: 
|   Backup files found:
|     /backup.zip
|     /config.bak
|_    /database.sql.old
```

---

## 📂 5. WebDAV Enumeration
```bash
# Basic WebDAV discovery
nmap --script http-enum -sV -p 80,443 <target-ip>

# Specific WebDAV scanning
nmap --script http-webdav-scan -p 80,443 <target-ip>

# WebDAV methods testing
nmap --script http-methods --script-args http-methods.url-path=/webdav/ <target-ip>

# Complete WebDAV assessment
nmap --script http-webdav-scan,http-methods,http-enum -p 80,443 <target-ip>

# Common WebDAV paths testing
for path in /webdav/ /dav/ /uploads/ /_vti_bin/ /exchange/ /public/; do
  echo "Testing path: $path"
  nmap --script http-methods --script-args http-methods.url-path=$path <target-ip>
done
```
**Purpose:**

WebDAV (Web Distributed Authoring and Versioning) allows clients to edit and manage files on remote web servers. Poor WebDAV configuration can lead to:
- Unauthorized file uploads
- Web shell deployment
- Sensitive file access
- Remote code execution

**INE Lab Example - Complete WebDAV Assessment:**
```bash
# Step 1: Initial discovery
nmap -sV -p 80 demo.ine.local

# Step 2: Directory enumeration
nmap --script http-enum -p 80 demo.ine.local

# Step 3: WebDAV-specific scanning
nmap --script http-webdav-scan -p 80 demo.ine.local

# Step 4: Methods testing on WebDAV directory
nmap --script http-methods --script-args http-methods.url-path=/webdav/ demo.ine.local
```
**Expected Complete Output:**
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0

| http-enum: 
|_  /webdav/: Potentially interesting folder (401 Unauthorized)

| http-webdav-scan: 
|   Server Type: Microsoft-IIS/10.0
|   WebDAV type: Unknown
|   Directory: /webdav/
|_  WebDAV: enabled

| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE COPY PROPFIND LOCK UNLOCK
|_  Potentially risky methods: TRACE COPY PROPFIND LOCK UNLOCK

|_http-server-header: Microsoft-IIS/10.0
```

**Analysis:**
- **WebDAV is enabled** on `/webdav/` directory
- **Authentication required** (401 status)
- **WebDAV methods available**: COPY, PROPFIND, LOCK, UNLOCK
- **Microsoft IIS 10.0** hosting the WebDAV service

**Next Steps for WebDAV:**
1. Attempt to find credentials for WebDAV access
2. Test for anonymous upload capabilities
3. Try uploading malicious files if access is gained
4. Check for file execution capabilities

---

## 🔐 6. SSL/TLS and HTTPS Testing
```bash
# SSL certificate information
nmap --script ssl-cert -p 443 <target-ip>

# SSL/TLS cipher enumeration
nmap --script ssl-enum-ciphers -p 443 <target-ip>

# SSL vulnerabilities scanning
nmap --script ssl-* -p 443 <target-ip>

# HTTPS-specific HTTP tests
nmap --script http-enum,http-title --script-args http.useragent="Mozilla/5.0" -p 443 <target-ip>

# Combined SSL and HTTP assessment
nmap --script ssl-cert,ssl-enum-ciphers,http-title,http-server-header -p 443 <target-ip>
```
**Purpose:**

SSL/TLS testing evaluates:
- Certificate validity and configuration
- Encryption strength and cipher suites
- SSL/TLS vulnerabilities
- HTTPS-specific web content

**Sample SSL Output:**
```bash
| ssl-cert: 
|   Subject: commonName=demo.ine.local
|   Issuer: commonName=demo.ine.local
|   Public Key type: rsa
|   Public Key bits: 2048
|   Signature Algorithm: sha256WithRSAEncryption
|   Not valid before: 2023-01-01T00:00:00
|_  Not valid after:  2024-01-01T00:00:00

| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 - A
|_      TLS_RSA_WITH_AES_256_CBC_SHA256 - A
```

---

## 🏗️ 7. Content Management Systems and Frameworks
```bash
# WordPress enumeration
nmap --script http-wordpress-enum -p 80,443 <target-ip>

# Drupal enumeration
nmap --script http-drupal-enum -p 80,443 <target-ip>

# Joomla enumeration
nmap --script http-joomla-brute -p 80,443 <target-ip>

# General CMS detection
nmap --script http-generator -p 80,443 <target-ip>

# Web framework detection
nmap --script http-title,http-server-header,http-generator -p 80,443 <target-ip>

# Technology stack identification
nmap --script http-php-version,http-asp-session-id-dir-traversal -p 80,443 <target-ip>
```
**Purpose:**

CMS and framework detection helps identify:
- Specific web applications and their versions
- Known vulnerabilities in web platforms
- Default configurations and common attack vectors
- Plugin and theme vulnerabilities

---

## 🔑 8. Authentication Testing
```bash
# Default credentials testing
nmap --script http-default-accounts -p 80,443 <target-ip>

# Basic authentication brute force
nmap --script http-brute -p 80,443 <target-ip>

# Form-based authentication testing
nmap --script http-form-brute -p 80,443 <target-ip>

# WordPress/CMS specific authentication
nmap --script http-wordpress-brute -p 80,443 <target-ip>

# Custom authentication brute force
nmap --script http-brute --script-args userdb=users.txt,passdb=passwords.txt -p 80 <target-ip>

# Authentication method detection
nmap --script http-auth-finder -p 80,443 <target-ip>
```
**Purpose:**

Authentication testing identifies:
- Default or weak credentials
- Authentication mechanisms in use
- Brute force attack possibilities
- Session management vulnerabilities

**Sample Authentication Output:**
```bash
| http-default-accounts: 
|   admin:admin - Valid credentials found
|_  guest:guest - Valid credentials found

| http-brute: 
|   Accounts: 
|     admin:password - Valid credentials
|_    test:test123 - Valid credentials
```

---

## 🛠️ 9. Advanced HTTP Enumeration

### Custom Headers and User Agents
```bash
# Custom user agent
nmap --script http-enum --script-args http.useragent="Custom Scanner 1.0" -p 80 <target-ip>

# Custom headers for bypass attempts
nmap --script http-enum --script-args http.header="X-Forwarded-For: 127.0.0.1" -p 80 <target-ip>

# Multiple custom headers
nmap --script http-* --script-args 'http.header="X-Real-IP: 127.0.0.1","X-Forwarded-For: 127.0.0.1"' -p 80 <target-ip>

# Request method modification
nmap --script http-methods --script-args http.max-cache-size=1000000 -p 80 <target-ip>
```

### Performance and Timing Optimization
```bash
# Fast HTTP enumeration
nmap -T4 --script http-enum,http-title -p 80,443 <target-ip>

# Thorough but slow enumeration
nmap -T2 --script http-* -p 80,443 <target-ip>

# Parallel HTTP scanning with custom timing
nmap --script http-enum --min-parallelism 10 --max-parallelism 20 -p 80,443 <target-list>

# Custom timeout settings
nmap --script http-enum --script-args http.timeout=10,http.pipeline=5 -p 80 <target-ip>
```

---

## 🎯 Complete HTTP Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: HTTP port discovery
nmap -sS -p 80,443,8080,8443,8000,8888,9000 <target>

# Step 2: Service version detection
nmap -sV -p 80,443,8080 <target>

# Step 3: Basic HTTP information gathering
nmap --script http-title,http-server-header -p 80,443,8080 <target>
```

### Phase 2: Directory and Content Discovery
```bash
# Step 4: Directory enumeration
nmap --script http-enum -p 80,443,8080 <target>

# Step 5: HTTP methods testing
nmap --script http-methods -p 80,443,8080 <target>

# Step 6: WebDAV assessment
nmap --script http-webdav-scan,http-methods --script-args http-methods.url-path=/webdav/ -p 80,443 <target>
```

### Phase 3: Security Assessment
```bash
# Step 7: SSL/TLS assessment (for HTTPS)
nmap --script ssl-cert,ssl-enum-ciphers -p 443 <target>

# Step 8: Web vulnerability scanning
nmap --script http-vuln-* -p 80,443 <target>

# Step 9: Authentication testing
nmap --script http-default-accounts,http-brute --script-args brute.firstonly=true -p 80,443 <target>
```

---

## 🔁 All-in-One Scan

### Basic HTTP Discovery (no authentication)
```bash
nmap -sV -p 80,443,8080,8443 --script "http-title,http-server-header,http-enum,http-methods" <target-ip>
```

### WebDAV-Focused Assessment
```bash
nmap -p 80,443,8080 --script "http-enum,http-webdav-scan,http-methods" --script-args http-methods.url-path=/webdav/ <target-ip>
```

### Security Assessment Scan
```bash
nmap -p 80,443 --script "http-vuln-*,http-default-accounts,ssl-cert,ssl-enum-ciphers" <target-ip>
```

### Comprehensive HTTP Assessment
```bash
nmap --script http-*,ssl-* -p 80,443,8080,8443,8000,8888 <target-ip>
```

---

## 📊 Real Lab Examples

### Example 1: IIS WebDAV Server Assessment
```bash
# Target: Microsoft IIS with WebDAV enabled

# Service detection
nmap -sV -p 80 192.168.1.100

# Directory enumeration
nmap --script http-enum -p 80 192.168.1.100

# WebDAV testing
nmap --script http-webdav-scan,http-methods --script-args http-methods.url-path=/webdav/ 192.168.1.100

# Authentication testing
nmap --script http-default-accounts -p 80 192.168.1.100
```

### Example 2: WordPress Site Assessment
```bash
# Target: WordPress website

# Basic discovery
nmap -sV --script http-title,http-generator -p 80,443 192.168.1.200

# WordPress-specific enumeration
nmap --script http-wordpress-enum -p 80,443 192.168.1.200

# WordPress brute force (if authorized)
nmap --script http-wordpress-brute --script-args brute.firstonly=true -p 80 192.168.1.200
```

### Example 3: SSL/TLS Web Server Assessment
```bash
# Target: HTTPS server with potential SSL issues

# SSL certificate analysis
nmap --script ssl-cert -p 443 192.168.1.50

# SSL cipher assessment
nmap --script ssl-enum-ciphers -p 443 192.168.1.50

# SSL vulnerability scanning
nmap --script ssl-vuln-* -p 443 192.168.1.50

# HTTPS content enumeration
nmap --script http-enum,http-title -p 443 192.168.1.50
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Timeout** | Scripts hang or timeout | Use `--script-args http.timeout=10` |
| **Connection Refused** | `Connection refused` | Check if web service is running |
| **SSL Handshake Failed** | SSL errors on HTTPS | Use specific SSL script arguments |
| **Empty Results** | No directories found | Try different base paths with `http-enum.basepath` |
| **Authentication Required** | 401/403 errors | Use authentication scripts or provide credentials |

### Common Error Messages:
- `http: could not connect` → Web service not available
- `http-enum: nothing found` → No interesting directories discovered
- `ssl: handshake failed` → SSL/TLS configuration issues
- `401 Unauthorized` → Authentication required for access
- `403 Forbidden` → Access denied by server policy

### Performance Optimization:
```bash
# Faster HTTP scanning
nmap -T4 --script http-enum,http-title -p 80,443 <target>

# Optimized for large networks
nmap --script http-enum --min-hostgroup 5 --max-parallelism 10 -p 80 <target-list>

# Custom timeout and cache settings
nmap --script http-* --script-args http.timeout=5,http.max-cache-size=1000000 -p 80,443 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **Web application testing can impact availability** - be cautious with timing
- **Follow responsible disclosure** if vulnerabilities are found

### Operational Security (OpSec):
- **HTTP requests generate extensive logs** - be aware of detection
- **Directory brute forcing is noisy** - consider using stealth techniques
- **Authentication brute force can lock accounts** - use with caution
- **SSL/TLS testing is generally safe** but may trigger security alerts

### Detection Indicators:
Target systems may detect HTTP enumeration through:
- **Web Logs**: Unusual request patterns and user agents
- **Security Tools**: WAF (Web Application Firewall) blocking
- **Monitoring Systems**: Automated vulnerability scanning detection
- **Failed Authentication**: Multiple login attempts triggering alerts

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| WebDAV with Upload | 🔴 Critical | Disable or secure WebDAV access |
| Default Credentials | 🔴 Critical | Change all default passwords |
| Weak SSL/TLS Ciphers | 🟠 High | Update cipher configuration |
| Directory Listing | 🟡 Medium | Disable directory browsing |
| Information Disclosure | 🟡 Medium | Remove verbose error messages |

---

## 🎯 eJPT Focus Points

### Essential HTTP Commands for eJPT:
```bash
# Basic HTTP discovery
nmap --script http-enum -sV -p 80,443 <target>

# WebDAV identification and testing
nmap --script http-webdav-scan,http-methods --script-args http-methods.url-path=/webdav/ -p 80 <target>

# Quick web server assessment
nmap --script http-title,http-server-header -p 80,443 <target>

# HTTP methods testing
nmap --script http-methods -p 80,443 <target>
```

### Key HTTP Assessment Areas for eJPT:
1. **Directory Discovery**: Use `http-enum` for finding hidden directories
2. **WebDAV Testing**: Check for file upload capabilities
3. **HTTP Methods**: Test for dangerous methods like PUT, DELETE
4. **Server Information**: Extract server versions and technology stack
5. **SSL/TLS Assessment**: Evaluate HTTPS security configuration
6. **Authentication**: Test for default credentials and weak authentication

### Common eJPT HTTP Scenarios:
- **WebDAV Upload**: Finding WebDAV directories for file upload attacks
- **Directory Traversal**: Accessing sensitive files through path manipulation
- **Default Credentials**: Logging into admin interfaces with default passwords
- **Information Disclosure**: Finding backup files or configuration files
- **SSL Vulnerabilities**: Exploiting weak SSL/TLS configurations

### Critical HTTP Information for Further Testing:
- **Directory Structure**: Hidden directories and files for further exploration
- **WebDAV Capabilities**: File upload possibilities for payload deployment
- **Authentication Mechanisms**: Login interfaces for credential attacks
- **Server Technology**: Software versions for vulnerability research
- **SSL/TLS Configuration**: Encryption weaknesses for man-in-the-middle attacks

---

## 🔗 Integration with Other Tools

After Nmap HTTP enumeration, follow up with:
- **Gobuster/Dirb**: Advanced directory brute forcing
- **Nikto**: Comprehensive web vulnerability scanning
- **Burp Suite**: Manual web application testing and proxy
- **davtest**: WebDAV upload testing and exploitation
- **cadaver**: WebDAV client for file management
- **sqlmap**: SQL injection testing and exploitation
- **wfuzz**: Web application fuzzer for parameter discovery

### HTTP Security Testing Workflow:
```bash
# 1. Nmap HTTP discovery and enumeration
nmap --script http-*,ssl-* -p 80,443,8080 <target>

# 2. Advanced directory discovery
gobuster dir -u http://<target> -w /usr/share/wordlists/dirb/common.txt

# 3. WebDAV testing (if found)
davtest -url http://<target>/webdav/

# 4. Vulnerability scanning
nikto -h http://<target>

# 5. Manual testing with proxy
# Configure Burp Suite and browse the application
```

---

## 📝 Output and Documentation

### Saving HTTP Assessment Results:
```bash
# Comprehensive HTTP assessment
nmap --script http-*,ssl-* -oA http_enumeration -p 80,443,8080,8443 <target>

# WebDAV-specific assessment
nmap --script http-webdav-scan,http-methods,http-enum -oN webdav_scan.txt -p 80 <target>

# SSL/TLS assessment
nmap --script ssl-cert,ssl-enum-ciphers -oX ssl_assessment.xml -p 443 <target>
```

### Results Processing and Analysis:
```bash
# Extract discovered directories
grep -E "Potentially interesting|directory" http_enumeration.nmap

# Find WebDAV servers
grep -i "webdav" http_enumeration.nmap

# Identify default credentials
grep -A 5 "Valid credentials" http_enumeration.nmap

# Extract SSL issues
grep -E "(weak|vulnerable|deprecated)" http_enumeration.nmap
```

---

## 📖 References and Further Reading

### Official Documentation:
- [Nmap NSE Scripts](https://nmap.org/nsedoc/scripts/) - Complete NSE script reference
- [HTTP Protocol RFC 7230](https://tools.ietf.org/html/rfc7230) - HTTP specification
- [WebDAV RFC 4918](https://tools.ietf.org/html/rfc4918) - WebDAV protocol specification

### Security Research:
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) - Web application security testing
- [SANS Web Application Security](https://www.sans.org/cyber-security-courses/web-app-penetration-testing-ethical-hacking/) - Professional web testing training

### Related Tools:
- [Gobuster](https://github.com/OJ/gobuster) - Directory/file brute-forcer
- [Nikto](https://github.com/sullo/nikto) - Web server scanner
- [Burp Suite](https://portswigger.net/burp) - Web application testing platform

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Scope**: Complete HTTP/HTTPS Service Enumeration Guide
