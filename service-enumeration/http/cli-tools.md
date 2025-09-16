# 🛠️ HTTP CLI Tools - Command Line Web Enumeration

This section covers various command-line tools for HTTP service enumeration and web application testing.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Tools Overview](#-tools-overview)
- [Prerequisites](#-prerequisites)
- [HTTP Header Analysis](#-1-http-header-analysis)
- [Response Inspection](#-2-response-inspection)
- [SSL/TLS Analysis](#-3-ssltls-analysis)
- [Directory Enumeration](#-4-directory-enumeration)
- [Web Technology Detection](#-5-web-technology-detection)
- [Authentication Testing](#-6-authentication-testing)
- [Advanced Techniques](#-7-advanced-techniques)
- [Output Analysis](#-8-output-analysis-and-interpretation)
- [Integration with Other Tools](#-9-integration-with-other-tools)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)
- [References](#-references)

---

## 🎯 Purpose

HTTP CLI tools help you:
- Analyze HTTP headers and response codes
- Inspect web server configurations and versions
- Test SSL/TLS implementations and certificates
- Enumerate directories and hidden resources
- Detect web technologies and frameworks
- Test authentication mechanisms
- Identify security misconfigurations
- Perform reconnaissance on web applications

---

## 🗂️ Tools Overview

| Tool | Purpose | Output Format | Authentication |
|------|---------|---------------|----------------|
| `curl` | HTTP client for requests/responses | Raw HTTP | Built-in support |
| `wget` | File download and mirroring | File-based | Basic auth |
| `openssl` | SSL/TLS analysis and testing | Certificate details | N/A |
| `httpie` | User-friendly HTTP client | JSON/formatted | OAuth support |
| `nc` (netcat) | Raw HTTP connections | Plain text | Manual |
| `telnet` | Manual HTTP testing | Interactive | Manual |

---

## 📋 Prerequisites

### Tool Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install curl wget openssl netcat-openbsd telnet httpie

# CentOS/RHEL
sudo yum install curl wget openssl nc telnet
pip install httpie

# macOS
brew install curl wget openssl netcat httpie

# Verify installations
curl --version
wget --version
openssl version
```

### Network Requirements:
- HTTP/HTTPS service running on target
- Network connectivity to target
- DNS resolution for domain names

### Quick Service Check:
```bash
# Test HTTP connectivity
curl -I http://<target>/

# Test HTTPS connectivity
curl -I https://<target>/

# Basic port check
nc -zv <target> 80 443
```

---

## 🔧 HTTP CLI Enumeration

## 🏷️ 1. HTTP Header Analysis

### Basic Header Inspection:
```bash
curl -I http://<target>/
```

**📌 INE Lab Example:**
```bash
curl -I http://demo.ine.local/
```

**📸 Sample Output:**
```bash
HTTP/1.1 200 OK
Date: Wed, 17 Sep 2025 12:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 10932
Connection: keep-alive
X-Powered-By: PHP/7.4.3
```

### Advanced Header Analysis:
```bash
# Include request headers
curl -v http://<target>/

# Custom headers
curl -H "User-Agent: CustomBot/1.0" -I http://<target>/

# Multiple headers
curl -H "Accept: application/json" -H "X-Forwarded-For: 127.0.0.1" -I http://<target>/
```

**📸 Verbose Output:**
```bash
* Connected to demo.ine.local (192.168.1.100) port 80 (#0)
> GET / HTTP/1.1
> Host: demo.ine.local
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Wed, 17 Sep 2025 12:00:00 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Content-Type: text/html; charset=UTF-8
< Set-Cookie: PHPSESSID=abc123; path=/
< X-Powered-By: PHP/7.4.3
< Content-Length: 10932
```

### Security Header Analysis:
```bash
#!/bin/bash
# Security header checker

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== HTTP Security Headers Analysis ==="
echo "Target: $TARGET"
echo

# Get headers
HEADERS=$(curl -I -s "$TARGET" 2>/dev/null)

echo "=== Response Headers ==="
echo "$HEADERS"
echo

echo "=== Security Headers Check ==="
security_headers=(
    "Strict-Transport-Security"
    "Content-Security-Policy"
    "X-Frame-Options"
    "X-Content-Type-Options"
    "X-XSS-Protection"
    "Referrer-Policy"
    "Permissions-Policy"
)

for header in "${security_headers[@]}"; do
    if echo "$HEADERS" | grep -qi "$header"; then
        echo "✓ $header: Present"
        echo "$HEADERS" | grep -i "$header"
    else
        echo "✗ $header: Missing"
    fi
done
```

---

## 🔍 2. Response Inspection

### Full Response Analysis:
```bash
curl http://<target>/
```

### Response with Timing:
```bash
curl -w "%{time_total}\n" http://<target>/
```

### Detailed Timing Information:
```bash
curl -w "DNS: %{time_namelookup}s\nConnect: %{time_connect}s\nSSL: %{time_appconnect}s\nTotal: %{time_total}s\nSize: %{size_download} bytes\n" -o /dev/null -s http://<target>/
```

**📸 Sample Timing Output:**
```bash
DNS: 0.001s
Connect: 0.002s
SSL: 0.045s
Total: 0.156s
Size: 10932 bytes
```

### Response Code Testing:
```bash
#!/bin/bash
# HTTP response code checker

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== HTTP Response Code Analysis ==="

# Common paths to test
paths=(
    "/"
    "/admin"
    "/login"
    "/robots.txt"
    "/sitemap.xml"
    "/.htaccess"
    "/config.php"
    "/test.php"
)

for path in "${paths[@]}"; do
    response_code=$(curl -s -o /dev/null -w "%{http_code}" "$TARGET$path")
    case $response_code in
        200) echo "✓ $TARGET$path - $response_code (OK)" ;;
        301|302|303|307|308) echo "→ $TARGET$path - $response_code (Redirect)" ;;
        401) echo "🔒 $TARGET$path - $response_code (Unauthorized)" ;;
        403) echo "🚫 $TARGET$path - $response_code (Forbidden)" ;;
        404) echo "✗ $TARGET$path - $response_code (Not Found)" ;;
        500) echo "💥 $TARGET$path - $response_code (Server Error)" ;;
        *) echo "? $TARGET$path - $response_code (Unknown)" ;;
    esac
done
```

### HTTP Methods Testing:
```bash
# Test different HTTP methods
for method in GET POST PUT DELETE OPTIONS HEAD TRACE; do
    echo "Testing $method:"
    curl -X $method -I http://<target>/ 2>/dev/null | head -1
done
```

---

## 🔐 3. SSL/TLS Analysis

### Certificate Information:
```bash
openssl s_client -connect <target>:443 -servername <target>
```

**📌 INE Lab Example:**
```bash
openssl s_client -connect demo.ine.local:443 -servername demo.ine.local
```

### Certificate Details Extraction:
```bash
# Get certificate details
echo | openssl s_client -connect <target>:443 -servername <target> 2>/dev/null | openssl x509 -noout -text

# Certificate expiration
echo | openssl s_client -connect <target>:443 -servername <target> 2>/dev/null | openssl x509 -noout -dates

# Certificate subject
echo | openssl s_client -connect <target>:443 -servername <target> 2>/dev/null | openssl x509 -noout -subject
```

**📸 Certificate Information:**
```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 12345678901234567890
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = Let's Encrypt Authority X3
        Validity
            Not Before: Sep  1 12:00:00 2025 GMT
            Not After : Dec  1 12:00:00 2025 GMT
        Subject: CN = demo.ine.local
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
```

### SSL/TLS Security Testing:
```bash
#!/bin/bash
# SSL/TLS security assessment

TARGET=$1
PORT=${2:-443}

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target> [port]"
    exit 1
fi

echo "=== SSL/TLS Security Assessment ==="
echo "Target: $TARGET:$PORT"
echo

# Test connection
echo "Testing connection..."
if echo | timeout 5 openssl s_client -connect "$TARGET:$PORT" >/dev/null 2>&1; then
    echo "✓ SSL/TLS connection successful"
else
    echo "✗ SSL/TLS connection failed"
    exit 1
fi

# Get certificate info
echo
echo "=== Certificate Information ==="
cert_info=$(echo | openssl s_client -connect "$TARGET:$PORT" -servername "$TARGET" 2>/dev/null)

# Extract key information
echo "$cert_info" | openssl x509 -noout -subject -dates -issuer 2>/dev/null

# Check for weak ciphers
echo
echo "=== Cipher Testing ==="
weak_ciphers=("RC4" "DES" "3DES" "MD5")
for cipher in "${weak_ciphers[@]}"; do
    if echo | openssl s_client -connect "$TARGET:$PORT" -cipher "$cipher" >/dev/null 2>&1; then
        echo "⚠ Weak cipher supported: $cipher"
    else
        echo "✓ Weak cipher rejected: $cipher"
    fi
done

# Protocol testing
echo
echo "=== Protocol Testing ==="
protocols=("ssl3" "tls1" "tls1_1" "tls1_2" "tls1_3")
for proto in "${protocols[@]}"; do
    if echo | openssl s_client -connect "$TARGET:$PORT" -$proto >/dev/null 2>&1; then
        case $proto in
            "ssl3"|"tls1"|"tls1_1") echo "⚠ Insecure protocol supported: $proto" ;;
            *) echo "✓ Secure protocol supported: $proto" ;;
        esac
    else
        echo "✗ Protocol not supported: $proto"
    fi
done
```

---

## 📂 4. Directory Enumeration

### Basic Directory Discovery:
```bash
# Using curl for common directories
for dir in admin login wp-admin phpmyadmin dashboard; do
    response=$(curl -s -o /dev/null -w "%{http_code}" http://<target>/$dir/)
    echo "$dir: $response"
done
```

### Recursive wget Mirroring:
```bash
# Mirror website structure (careful with size)
wget -r -np -k --level=2 http://<target>/

# Dry run to see what would be downloaded
wget -r -np --spider --level=2 http://<target>/
```

### Custom Directory Bruteforce:
```bash
#!/bin/bash
# Simple directory brute-forcer

TARGET=$1
WORDLIST=${2:-"/usr/share/wordlists/dirb/common.txt"}

if [[ -z "$TARGET" || ! -f "$WORDLIST" ]]; then
    echo "Usage: $0 <target> [wordlist]"
    echo "Default wordlist: /usr/share/wordlists/dirb/common.txt"
    exit 1
fi

echo "=== Directory Enumeration ==="
echo "Target: $TARGET"
echo "Wordlist: $WORDLIST"
echo

found_dirs=()
while read -r dir; do
    if [[ ! -z "$dir" && ! "$dir" =~ ^# ]]; then
        response_code=$(curl -s -o /dev/null -w "%{http_code}" "$TARGET/$dir")
        case $response_code in
            200|301|302|403)
                echo "✓ /$dir ($response_code)"
                found_dirs+=("$dir")
                ;;
            *) ;;
        esac
    fi
done < "$WORDLIST"

echo
echo "Found directories: ${#found_dirs[@]}"
for dir in "${found_dirs[@]}"; do
    echo "  /$dir"
done
```

---

## 🔧 5. Web Technology Detection

### Server Technology Detection:
```bash
# Basic server detection
curl -I http://<target>/ | grep -i server

# X-Powered-By detection
curl -I http://<target>/ | grep -i x-powered-by

# Technology stack analysis
curl -s http://<target>/ | grep -i -E "(wordpress|drupal|joomla|php|asp|jsp)"
```

### Advanced Technology Detection:
```bash
#!/bin/bash
# Web technology fingerprinting

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Web Technology Detection ==="
echo "Target: $TARGET"
echo

# Get response headers and body
headers=$(curl -I -s "$TARGET" 2>/dev/null)
body=$(curl -s "$TARGET" 2>/dev/null)

echo "=== Server Information ==="
echo "$headers" | grep -i server
echo "$headers" | grep -i x-powered-by
echo "$headers" | grep -i x-aspnet-version

echo
echo "=== Framework Detection ==="

# WordPress
if echo "$body" | grep -qi "wp-content\|wordpress"; then
    echo "✓ WordPress detected"
    wp_version=$(echo "$body" | grep -o 'wp-includes.*ver=[0-9.]*' | head -1 | cut -d= -f2 | cut -d\' -f1)
    [[ ! -z "$wp_version" ]] && echo "  Version: $wp_version"
fi

# Drupal
if echo "$body" | grep -qi "drupal\|sites/default"; then
    echo "✓ Drupal detected"
fi

# Joomla
if echo "$body" | grep -qi "joomla\|option=com_"; then
    echo "✓ Joomla detected"
fi

# PHP
if echo "$headers" | grep -qi "php" || echo "$body" | grep -qi "phpsessid"; then
    echo "✓ PHP detected"
    php_version=$(echo "$headers" | grep -i x-powered-by | grep -o 'PHP/[0-9.]*' | cut -d/ -f2)
    [[ ! -z "$php_version" ]] && echo "  Version: $php_version"
fi

# ASP.NET
if echo "$headers" | grep -qi "asp\|x-aspnet"; then
    echo "✓ ASP.NET detected"
    aspnet_version=$(echo "$headers" | grep -i x-aspnet-version | cut -d: -f2 | tr -d ' ')
    [[ ! -z "$aspnet_version" ]] && echo "  Version: $aspnet_version"
fi

# JavaScript frameworks
echo
echo "=== JavaScript Frameworks ==="
js_frameworks=("jquery" "angular" "react" "vue" "bootstrap")
for framework in "${js_frameworks[@]}"; do
    if echo "$body" | grep -qi "$framework"; then
        echo "✓ $framework detected"
    fi
done

echo
echo "=== Security Features ==="
if echo "$headers" | grep -qi "content-security-policy"; then
    echo "✓ Content Security Policy enabled"
fi

if echo "$headers" | grep -qi "x-frame-options"; then
    echo "✓ X-Frame-Options enabled"
fi

if echo "$headers" | grep -qi "strict-transport-security"; then
    echo "✓ HSTS enabled"
fi
```

---

## 🔐 6. Authentication Testing

### Basic Authentication Testing:
```bash
# Test basic auth
curl -u username:password http://<target>/admin/

# Test with empty credentials
curl -u : http://<target>/admin/

# Test common credentials
curl -u admin:admin http://<target>/admin/
curl -u admin:password http://<target>/admin/
```

### Form-based Authentication:
```bash
# POST login data
curl -d "username=admin&password=admin" -X POST http://<target>/login

# Include cookies
curl -c cookies.txt -d "username=admin&password=admin" -X POST http://<target>/login

# Use cookies for authenticated request
curl -b cookies.txt http://<target>/dashboard
```

### Authentication Bypass Testing:
```bash
#!/bin/bash
# Authentication bypass testing

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Authentication Bypass Testing ==="
echo "Target: $TARGET"
echo

# SQL injection attempts
sqli_payloads=(
    "admin' OR '1'='1"
    "admin'--"
    "admin' /*"
    "' OR 1=1--"
)

echo "Testing SQL injection payloads..."
for payload in "${sqli_payloads[@]}"; do
    response=$(curl -s -d "username=$payload&password=test" -X POST "$TARGET/login")
    if echo "$response" | grep -qi "dashboard\|welcome\|success"; then
        echo "⚠ Potential SQLi: $payload"
    fi
done

# Directory traversal
echo
echo "Testing directory traversal..."
traversal_paths=("../admin" "../../admin" "../../../etc/passwd")
for path in "${traversal_paths[@]}"; do
    response_code=$(curl -s -o /dev/null -w "%{http_code}" "$TARGET/$path")
    if [[ "$response_code" == "200" ]]; then
        echo "⚠ Potential path traversal: $path"
    fi
done
```

---

## 🚀 7. Advanced Techniques

### HTTP/2 Support Testing:
```bash
# Test HTTP/2 support
curl --http2 -I https://<target>/

# HTTP/2 with prior knowledge
curl --http2-prior-knowledge -I http://<target>/
```

### Cookie Analysis:
```bash
#!/bin/bash
# Cookie security analysis

TARGET=$1
if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

echo "=== Cookie Security Analysis ==="
echo "Target: $TARGET"
echo

# Get cookies
cookies=$(curl -I -s "$TARGET" 2>/dev/null | grep -i set-cookie)

if [[ -z "$cookies" ]]; then
    echo "No cookies found"
    exit 0
fi

echo "Found cookies:"
echo "$cookies"
echo

# Analyze cookie security
echo "Cookie Security Analysis:"
while IFS= read -r cookie; do
    cookie_name=$(echo "$cookie" | cut -d: -f2 | cut -d= -f1 | tr -d ' ')
    if [[ ! -z "$cookie_name" ]]; then
        echo "Cookie: $cookie_name"
        
        # Check for security flags
        if echo "$cookie" | grep -qi "secure"; then
            echo "  ✓ Secure flag present"
        else
            echo "  ✗ Secure flag missing"
        fi
        
        if echo "$cookie" | grep -qi "httponly"; then
            echo "  ✓ HttpOnly flag present"
        else
            echo "  ✗ HttpOnly flag missing"
        fi
        
        if echo "$cookie" | grep -qi "samesite"; then
            echo "  ✓ SameSite flag present"
        else
            echo "  ✗ SameSite flag missing"
        fi
        
        echo
    fi
done <<< "$cookies"
```

### Load Testing:
```bash
#!/bin/bash
# Simple load testing with curl

TARGET=$1
REQUESTS=${2:-10}
CONCURRENT=${3:-2}

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target> [requests] [concurrent]"
    exit 1
fi

echo "=== Load Testing ==="
echo "Target: $TARGET"
echo "Requests: $REQUESTS"
echo "Concurrent: $CONCURRENT"
echo

# Sequential timing
echo "Sequential requests:"
start_time=$(date +%s.%N)
for ((i=1; i<=REQUESTS; i++)); do
    response_time=$(curl -w "%{time_total}" -s -o /dev/null "$TARGET")
    echo "Request $i: ${response_time}s"
done
end_time=$(date +%s.%N)
total_time=$(echo "$end_time - $start_time" | bc)
echo "Total time: ${total_time}s"

# Parallel requests
echo
echo "Parallel requests ($CONCURRENT concurrent):"
start_time=$(date +%s.%N)
for ((i=1; i<=REQUESTS; i+=CONCURRENT)); do
    for ((j=0; j<CONCURRENT && i+j<=REQUESTS; j++)); do
        curl -w "Request $((i+j)): %{time_total}s\n" -s -o /dev/null "$TARGET" &
    done
    wait
done
end_time=$(date +%s.%N)
total_time=$(echo "$end_time - $start_time" | bc)
echo "Total time: ${total_time}s"
```

---

## 🔍 8. Output Analysis and Interpretation

### HTTP Status Code Analysis:

| Code | Meaning | Security Implication |
|------|---------|---------------------|
| `200` | OK | Resource accessible |
| `301/302` | Redirect | May reveal internal structure |
| `401` | Unauthorized | Authentication required |
| `403` | Forbidden | Resource exists but access denied |
| `404` | Not Found | Resource doesn't exist |
| `500` | Server Error | Application error, potential info disclosure |

### Header Analysis:

**Server Information:**
```bash
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/7.4.3
```
- **Analysis**: Apache web server version 2.4.41 on Ubuntu
- **PHP Version**: 7.4.3 - check for vulnerabilities
- **Action**: Research known CVEs for these versions

**Security Headers:**
```bash
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
```
- **HSTS**: Forces HTTPS connections
- **CSP**: Prevents XSS attacks
- **X-Frame-Options**: Prevents clickjacking

### Response Timing Analysis:

**Normal Response Times:**
- **Static files**: < 100ms
- **Dynamic pages**: 100-500ms
- **Database queries**: 200-1000ms
- **Slow responses**: > 2000ms (potential DoS or heavy processing)

### Certificate Analysis:

**Certificate Security:**
```bash
Subject: CN=demo.ine.local
Issuer: CN=Let's Encrypt Authority X3
Not After: Dec  1 12:00:00 2025 GMT
```
- **Valid Certificate**: Issued by trusted CA
- **Expiration**: Check renewal dates
- **Subject**: Verify domain matches

---

## 🔗 9. Integration with Other Tools

### With Nmap:
```bash
# Nmap HTTP enumeration
nmap -p 80,443 --script http-enum <target>

# Follow up with curl analysis
curl -I http://<target>/
```

### With Burp Suite:
```bash
# Generate requests for Burp analysis
curl -v http://<target>/ --proxy http://127.0.0.1:8080

# Export curl commands from Burp
# Right-click request → Copy as curl command
```

### With Gobuster:
```bash
# Directory enumeration with gobuster
gobuster dir -u http://<target>/ -w /usr/share/wordlists/dirb/common.txt

# Validate findings with curl
curl -I http://<target>/admin/
```

### Pipeline Integration:
```bash
#!/bin/bash
# HTTP enumeration pipeline

TARGET=$1
OUTPUT_DIR="http_enum_$(date +%Y%m%d_%H%M%S)"

if [[ -z "$TARGET" ]]; then
    echo "Usage: $0 <target>"
    exit 1
fi

mkdir -p "$OUTPUT_DIR"
echo "=== HTTP Enumeration Pipeline ==="
echo "Target: $TARGET"
echo "Output: $OUTPUT_DIR"
echo

# Step 1: Basic reconnaissance
echo "Step 1: Basic reconnaissance"
curl -I "$TARGET" > "$OUTPUT_DIR/headers.txt" 2>&1
curl -s "$TARGET" > "$OUTPUT_DIR/index.html" 2>&1

# Step 2: Security headers check
echo "Step 2: Security analysis"
./security_headers.sh "$TARGET" > "$OUTPUT_DIR/security_analysis.txt"

# Step 3: Technology detection
echo "Step 3: Technology detection"
./tech_detection.sh "$TARGET" > "$OUTPUT_DIR/technology.txt"

# Step 4: SSL analysis (if HTTPS)
if [[ "$TARGET" =~ https ]]; then
    echo "Step 4: SSL analysis"
    ./ssl_analysis.sh "${TARGET#https://}" > "$OUTPUT_DIR/ssl_analysis.txt"
fi

# Step 5: Directory enumeration
echo "Step 5: Directory enumeration"
if command -v gobuster >/dev/null; then
    gobuster dir -u "$TARGET" -w /usr/share/wordlists/dirb/common.txt -o "$OUTPUT_DIR/directories.txt" -q
else
    ./simple_dir_enum.sh "$TARGET" > "$OUTPUT_DIR/directories.txt"
fi

# Step 6: Generate report
echo "Step 6: Generating report"
cat > "$OUTPUT_DIR/report.md" << EOF
# HTTP Enumeration Report
**Target:** $TARGET  
**Date:** $(date)

## Headers
\`\`\`
$(cat "$OUTPUT_DIR/headers.txt")
\`\`\`

## Technology Stack
$(cat "$OUTPUT_DIR/technology.txt")

## Security Analysis
$(cat "$OUTPUT_DIR/security_analysis.txt")

## Discovered Directories
$(cat "$OUTPUT_DIR/directories.txt" | grep -E "✓|200|301|403" || echo "None found")
EOF

echo "Enumeration complete. Report saved to $OUTPUT_DIR/report.md"
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **SSL Handshake Failure** | `SSL_ERROR_*` messages | Check SSL/TLS configuration |
| **Connection Timeout** | Requests hang indefinitely | Check network connectivity and firewall |
| **Certificate Errors** | Certificate verification failed | Use `curl -k` to ignore certificate errors |
| **HTTP 403 Errors** | Forbidden responses | Check User-Agent, IP restrictions |
| **Large Response Bodies** | Slow downloads | Use `curl -I` for headers only |

### Debugging Commands:
```bash
# Verbose curl output
curl -v http://<target>/

# Trace network connections
curl --trace-ascii trace.txt http://<target>/

# Test connectivity
ping <target>
telnet <target> 80

# DNS resolution
nslookup <target>
dig <target>
```

### Performance Optimization:
```bash
# Faster requests with keepalive
curl --keepalive-time 2 http://<target>/

# Limit response size
curl --max-filesize 1048576 http://<target>/

# Set timeouts
curl --connect-timeout 5 --max-time 10 http://<target>/

# Use HTTP/2 for better performance
curl --http2 https://<target>/
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Authorization Required**: Only test web applications you own or have explicit permission to test
- **Scope Compliance**: Respect testing boundaries and avoid production systems
- **Rate Limiting**: Don't overwhelm servers with excessive requests
- **Data Sensitivity**: Handle any discovered data responsibly

### Operational Security:
- **Request Logging**: Web servers log all HTTP requests with IP addresses
- **User-Agent Tracking**: Default curl user-agent is easily identifiable
- **SSL Certificate Logs**: Certificate Transparency logs record certificate requests
- **WAF Detection**: Web Application Firewalls may block or rate-limit requests

### Detection Indicators:
- **Access Logs**: Unusual request patterns or user agents
- **Security Tools**: Requests to security-related paths (/admin, /.env)
- **Error Rates**: High number of 404s from directory enumeration
- **Request Frequency**: Automated scanning patterns

### Defensive Countermeasures:
- **Web Application Firewall**: Deploy WAF to filter malicious requests
- **Rate Limiting**: Implement request rate limiting per IP
- **Security Headers**: Deploy comprehensive security headers
- **Access Controls**: Implement proper authentication and authorization
- **Monitoring**: Deploy comprehensive web application monitoring

### Risk Assessment:
| Activity | Detection Risk | Impact | Recommendation |
|----------|----------------|--------|----------------|
| Header Analysis | 🟢 Low | 🟢 Low | Generally safe for reconnaissance |
| Directory Enumeration | 🟠 High | 🟡 Medium | Use with caution, may trigger WAF |
| Authentication Testing | 🔴 High | 🔴 High | Only with explicit authorization |
| SSL Testing | 🟡 Medium | 🟢 Low | Safe but may be logged |

---

## 📖 References

### Official Documentation:
- [curl Manual](https://curl.se/docs/manual.html) - Comprehensive curl documentation
- [OpenSSL Documentation](https://www.openssl.org/docs/) - SSL/TLS testing reference
- [HTTP/1.1 Specification](https://tools.ietf.org/html/rfc7231) - RFC 7231 HTTP semantics

### Security Resources:
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) - Web application testing methodology
- [Mozilla Security Headers](https://infosec.mozilla.org/guidelines/web_security) - Security header best practices
- [SSL Labs Testing](https://www.ssllabs.com/ssltest/) - Online SSL testing tool

### Learning Resources:
- [HTTP Security Headers](https://securityheaders.com/) - Online security header checker
- [Web Application Hacker's Handbook](https://www.wiley.com/en-us/The+Web+Application+Hacker%27s+Handbook%3A+Finding+and+Exploiting+Security+Flaws%2C+2nd+Edition-p-9781118026472) - Comprehensive web security testing
- [Burp Suite Documentation](https://portswigger.net/burp/documentation) - Professional web application testing

---

## 📋 Quick Reference

### Essential Commands:
```bash
# Basic header inspection
curl -I http://<target>/

# Full response with timing
curl -w "%{time_total}\n" http://<target>/

# SSL certificate check
echo | openssl s_client -connect <target>:443 -servername <target>

# Security headers analysis
curl -I http://<target>/ | grep -i -E "(strict-transport|content-security|x-frame|x-content-type)"
```

### Key Headers to Analyze:
- **Server**: Web server software and version
- **X-Powered-By**: Backend technology (PHP, ASP.NET)
- **Set-Cookie**: Session management and security flags
- **Content-Security-Policy**: XSS protection
- **Strict-Transport-Security**: HTTPS enforcement

### HTTP Methods to Test:
```bash
# Standard methods
GET, POST, PUT, DELETE

# Extended methods
OPTIONS, HEAD, TRACE, PATCH

# WebDAV methods
PROPFIND, PROPPATCH, MKCOL, COPY, MOVE
```

### Response Code Priorities:
1. **200**: Accessible resources - enumerate further
2. **301/302**: Redirects - follow for hidden resources
3. **401**: Authentication required - test credentials
4. **403**: Forbidden - may indicate hidden resources
5. **500**: Server errors - potential information disclosure

### SSL/TLS Security Checks:
- Certificate validity and expiration
- Supported protocols (avoid SSLv3, TLSv1.0, TLSv1.1)
- Cipher strength (avoid RC4, DES, 3DES)
- Perfect Forward Secrecy support

### Technology Detection Patterns:
```bash
# WordPress indicators
wp-content, wp-includes, wp-admin

# Drupal indicators
sites/default, modules/, themes/

# PHP indicators
PHPSESSID, .php extensions

# ASP.NET indicators
ASPXAUTH, .aspx extensions, ViewState
```

### Security Testing Priorities:
1. **Authentication bypass** - SQLi, weak credentials
2. **Directory traversal** - ../../../etc/passwd
3. **Information disclosure** - error messages, stack traces
4. **Session management** - cookie security flags
5. **Input validation** - XSS, command injection

### File Extensions of Interest:
- **.env** - Environment configuration files
- **.git** - Git repository information
- **.htaccess** - Apache configuration
- **config.php** - Application configuration
- **backup.sql** - Database backups

### Common Attack Vectors to Test:
```bash
# SQL injection
admin' OR '1'='1--

# XSS
<script>alert('XSS')</script>

# Command injection
; cat /etc/passwd

# Path traversal
../../../etc/passwd

# LDAP injection
*)(&(objectClass=*
```

### Performance Benchmarks:
- **DNS Resolution**: < 50ms
- **TCP Connection**: < 100ms
- **SSL Handshake**: < 200ms
- **First Byte**: < 500ms
- **Full Page Load**: < 2000ms

### Cookie Security Flags:
- **Secure**: Cookie only sent over HTTPS
- **HttpOnly**: Cookie not accessible via JavaScript
- **SameSite**: CSRF protection (Strict/Lax/None)

### Time Estimates:
- **Basic header analysis**: 30 seconds
- **Technology detection**: 2-5 minutes
- **SSL analysis**: 1-3 minutes
- **Directory enumeration**: 5-30 minutes
- **Comprehensive assessment**: 30-60 minutes

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Author**: Penetration Testing Team

---

*Always ensure proper authorization before conducting HTTP security assessments.*
