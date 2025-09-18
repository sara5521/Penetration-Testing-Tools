# 🌐 DNS Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate DNS (Domain Name System) services and assess DNS infrastructure security for penetration testing and security assessments.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-dns-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [DNS Zone Transfer Testing](#-1-dns-zone-transfer-testing)
- [DNS Subdomain Discovery](#-2-dns-subdomain-and-host-discovery)
- [DNS Security Assessment](#-3-dns-security-assessment)
- [DNS Server Information](#-4-dns-server-information-gathering)
- [DNS Cache Analysis](#-5-dns-cache-snooping-and-analysis)
- [DNS Protocol Variations](#-6-dns-port-and-protocol-variations)
- [Advanced Techniques](#-7-advanced-dns-enumeration-techniques)
- [Complete Assessment Workflow](#-complete-dns-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from DNS services, such as:
- DNS zone transfer capabilities and data
- Subdomain and hostname enumeration
- DNS server software and version identification
- DNS security configurations and vulnerabilities
- Service record discovery (SRV, MX, etc.)
- DNS cache analysis and snooping
- DNS recursion and update capabilities
- Internal network infrastructure mapping
- DNS-based attack vector identification

---

## 🗂️ DNS Enumeration Scripts Summary

| # | Script                    | What it Does                             | Auth Required | Ports |
|---|---------------------------|------------------------------------------|---------------|-------|
| 1 | dns-zone-transfer         | Attempt DNS zone transfer (AXFR)        | ❌            | 53 |
| 2 | dns-brute                 | Brute force subdomain enumeration       | ❌            | 53 |
| 3 | dns-service-discovery     | Discover DNS service records             | ❌            | 53 |
| 4 | dns-recursion             | Test DNS recursion capability            | ❌            | 53 |
| 5 | dns-cache-snoop           | DNS cache snooping for reconnaissance    | ❌            | 53 |
| 6 | dns-update                | Test DNS dynamic update capability       | ❌            | 53 |
| 7 | dns-nsid                  | Extract DNS Name Server Identifier      | ❌            | 53 |
| 8 | dns-random-txid           | Test DNS transaction ID randomization    | ❌            | 53 |
| 9 | dns-random-srcport        | Test DNS source port randomization       | ❌            | 53 |
| 10 | dns-fuzz                  | DNS server fuzzing and fingerprinting   | ❌            | 53 |
| 11 | dns-srv-enum              | Enumerate DNS SRV records                | ❌            | 53 |
| 12 | dns-check-zone            | DNS zone consistency checking            | ❌            | 53 |

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has DNS service enabled** (port 53 UDP/TCP)
- **Proper authorization** to test the target
- **Network connectivity** to the target
- **Domain name information** for targeted enumeration

### Quick DNS Port Check:
```bash
# Check DNS UDP port
nmap -sU -p 53 <target-ip>

# Check DNS TCP port (for zone transfers)
nmap -sT -p 53 <target-ip>

# Check both UDP and TCP
nmap -sU -sT -p 53 <target-ip>
```

### Standard DNS Ports:
- **53**: DNS (UDP for queries, TCP for zone transfers)
- **853**: DNS over TLS (DoT)
- **443**: DNS over HTTPS (DoH) when applicable
- **5353**: mDNS (Multicast DNS)
- **8053**: Alternative DNS port

---

## 🔧 DNS Service Enumeration Commands

## 🏗️ 1. DNS Zone Transfer Testing
```bash
# Basic zone transfer attempt
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<domain> -p 53 <target-ip>

# Zone transfer with specific server
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com,dns-zone-transfer.server=<dns_server> -p 53 <target-ip>

# Multiple domain zone transfer testing
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com,dns-zone-transfer.domain=subdomain.example.com -p 53 <target-ip>

# Zone transfer with all discovered name servers
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com,dns-zone-transfer.addall=true -p 53 <target-ip>
```
**Purpose:**

DNS Zone Transfer (AXFR) is one of the most critical DNS enumeration techniques. It attempts to retrieve the complete DNS zone database from a DNS server, which can reveal:
- All subdomains and hostnames in the domain
- IP addresses of internal systems
- Network infrastructure layout
- Service locations (mail, web, FTP servers)
- Administrative contact information

**INE Lab Example:**
```bash
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=internal.company.com -p 53 demo.ine.local
```
**Sample Output (Successful Zone Transfer):**
```bash
PORT   STATE SERVICE
53/tcp open  domain

| dns-zone-transfer: 
| internal.company.com:
|   SOA   ns1.internal.company.com. admin.internal.company.com. 2024010101 10800 3600 604800 86400
|   NS    ns1.internal.company.com.
|   NS    ns2.internal.company.com.
|   A     web.internal.company.com.     192.168.1.100
|   A     mail.internal.company.com.    192.168.1.101
|   A     ftp.internal.company.com.     192.168.1.102
|   A     admin.internal.company.com.   192.168.1.103
|   A     database.internal.company.com. 192.168.1.104
|   CNAME intranet.internal.company.com. web.internal.company.com.
|   MX    mail.internal.company.com.    10
|_  TXT   internal.company.com.         "v=spf1 include:_spf.company.com ~all"
```
**Interpretation:**
- **Zone transfer successful** - major security vulnerability
- **Multiple internal systems discovered**: web, mail, FTP, admin, database servers
- **Internal IP range revealed**: 192.168.1.x network
- **Administrative email**: admin.internal.company.com
- **Email configuration**: SPF record indicates mail server setup

**Security Impact:**
- Complete network mapping achieved
- High-value targets identified (admin, database servers)
- Internal IP addressing scheme revealed
- Foundation for targeted attacks established

### Advanced Zone Transfer Testing:
```bash
# Zone transfer with custom timeout
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com,dns-zone-transfer.timeout=10 -p 53 <target-ip>

# Zone transfer with maximum records limit
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=example.com,dns-zone-transfer.maxrecords=1000 -p 53 <target-ip>

# Comprehensive zone transfer testing
nmap --script dns-zone-transfer,dns-nsid --script-args dns-zone-transfer.domain=example.com -p 53 <target-ip>
```

---

## 🕵️ 2. DNS Subdomain and Host Discovery
```bash
# Basic DNS brute force
nmap --script dns-brute --script-args dns-brute.domain=example.com -p 53 <target-ip>

# DNS brute force with custom wordlist
nmap --script dns-brute --script-args dns-brute.domain=example.com,dns-brute.hostlist=subdomains.txt -p 53 <target-ip>

# DNS brute force with thread control
nmap --script dns-brute --script-args dns-brute.domain=example.com,dns-brute.threads=10 -p 53 <target-ip>

# DNS brute force with specific record types
nmap --script dns-brute --script-args dns-brute.domain=example.com,dns-brute.type=A -p 53 <target-ip>
```
**Purpose:**

DNS brute forcing discovers subdomains that may not be publicly listed or linked, including:
- Development and staging environments
- Administrative interfaces
- Internal applications
- API endpoints
- Legacy systems

**INE Lab Example:**
```bash
nmap --script dns-brute --script-args dns-brute.domain=company.com -p 53 demo.ine.local
```
**Sample Output:**
```bash
| dns-brute: 
|   DNS Brute forcer
|   admin.company.com - 192.168.1.103
|   api.company.com - 192.168.1.105
|   dev.company.com - 192.168.1.106
|   ftp.company.com - 192.168.1.102
|   mail.company.com - 192.168.1.101
|   test.company.com - 192.168.1.107
|   web.company.com - 192.168.1.100
|_  www.company.com - 192.168.1.100
```
**Analysis:**
- **Administrative interface**: admin.company.com (high-value target)
- **Development environment**: dev.company.com (often less secured)
- **API endpoint**: api.company.com (potential for API vulnerabilities)
- **Test environment**: test.company.com (may contain sensitive test data)

### Service Record Discovery:
```bash
# DNS service discovery
nmap --script dns-service-discovery --script-args dns-service-discovery.domain=example.com -p 53 <target-ip>

# DNS SRV record enumeration
nmap --script dns-srv-enum --script-args dns-srv-enum.domain=example.com -p 53 <target-ip>

# DNS random subdomain testing
nmap --script dns-random-txid,dns-random-srcport -p 53 <target-ip>
```

---

## 🛡️ 3. DNS Security Assessment
```bash
# DNS cache snooping
nmap --script dns-cache-snoop --script-args dns-cache-snoop.domains={google.com,facebook.com,youtube.com} -p 53 <target-ip>

# Targeted cache snooping with specific domains
nmap --script dns-cache-snoop --script-args dns-cache-snoop.domains={internal.company.com,admin.company.com} -p 53 <target-ip>

# DNS cache snooping with mode specification
nmap --script dns-cache-snoop --script-args dns-cache-snoop.mode=timed -p 53 <target-ip>
```
**Purpose:**

DNS cache snooping reveals which domains have been recently queried by users on the network, potentially exposing:
- Internal domain names
- Frequently accessed external services
- User browsing patterns
- Network usage statistics

**Sample Cache Snooping Output:**
```bash
| dns-cache-snoop: 
|   Cache snooping results:
|     admin.company.com - Cached
|     internal.company.com - Cached
|     vpn.company.com - Cached
|     mail.google.com - Cached
|_    www.facebook.com - Not cached
```

### DNS Security Feature Testing:
```bash
# DNS recursion testing
nmap --script dns-recursion -p 53 <target-ip>

# DNS update testing
nmap --script dns-update --script-args dns-update.hostname=test.example.com,dns-update.ip=1.2.3.4 -p 53 <target-ip>

# DNS NSID (Name Server Identifier)
nmap --script dns-nsid -p 53 <target-ip>

# DNS client subnet testing
nmap --script dns-client-subnet-scan -p 53 <target-ip>
```

---

## 🔧 4. DNS Server Information Gathering
```bash
# DNS server version detection
nmap -sU -sV -p 53 <target-ip>

# DNS server banner and version
nmap --script banner,dns-nsid -p 53 <target-ip>

# DNS server software fingerprinting
nmap --script dns-fuzz -p 53 <target-ip>

# DNS server capability testing
nmap --script dns-recursion,dns-update -p 53 <target-ip>
```
**Purpose:**

DNS server information gathering reveals:
- DNS software type and version
- Server capabilities and features
- Potential vulnerabilities
- Configuration details

**Sample DNS Server Information:**
```bash
PORT   STATE SERVICE VERSION
53/udp open  domain  ISC BIND 9.11.4-P2 (Ubuntu Linux)

| dns-nsid: 
|_  bind.version: 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.8

| dns-recursion: 
|_  Recursion appears to be enabled
```

### DNS Configuration Analysis:
```bash
# DNS response analysis
nmap --script dns-random-txid,dns-random-srcport -p 53 <target-ip>

# DNS server load testing
nmap --script dns-fuzz --script-args dns-fuzz.max-threads=10 -p 53 <target-ip>

# DNS check zone consistency
nmap --script dns-check-zone --script-args dns-check-zone.domain=example.com -p 53 <target-ip>
```

---

## 🔍 5. DNS Cache Snooping and Analysis
```bash
# Basic cache snooping
nmap --script dns-cache-snoop --script-args dns-cache-snoop.domains={target1.com,target2.com} -p 53 <target-ip>

# Timed cache snooping for accuracy
nmap --script dns-cache-snoop --script-args dns-cache-snoop.mode=timed -p 53 <target-ip>

# Cache snooping with internal domains
nmap --script dns-cache-snoop --script-args dns-cache-snoop.domains={internal.company.local,admin.company.local} -p 53 <target-ip>
```
**Purpose:**

Cache snooping exploits DNS caching to determine:
- Which domains have been recently accessed
- Internal network domain names
- User activity patterns
- Potential target systems

---

## 📊 6. DNS Port and Protocol Variations
```bash
# Standard DNS UDP port 53
nmap -sU --script dns-* -p 53 <target-ip>

# DNS over TCP (for large responses)
nmap -sT --script dns-zone-transfer -p 53 <target-ip>

# DNS over TCP and UDP
nmap -sU -sT --script dns-recursion -p 53 <target-ip>

# DNS service on alternative ports
nmap --script dns-* -p 5353,8053,9953 <target-ip>
```

### DNS over HTTPS and TLS:
```bash
# DNS over HTTPS (DoH) - Port 443
nmap -sV -p 443 <target-ip> | grep -i dns

# DNS over TLS (DoT) - Port 853
nmap -sV -p 853 <target-ip>

# Secure DNS services
nmap --script ssl-enum-ciphers -p 853,443 <target-ip>
```

---

## 🛠️ 7. Advanced DNS Enumeration Techniques

### DNS Server Load and Stress Testing:
```bash
# DNS fuzzing for vulnerability discovery
nmap --script dns-fuzz --script-args dns-fuzz.max-threads=5 -p 53 <target-ip>

# DNS response time analysis
nmap --script dns-cache-snoop --script-args dns-cache-snoop.mode=timed -p 53 <target-ip>

# DNS server capability enumeration
nmap --script dns-nsid,dns-recursion,dns-update -p 53 <target-ip>
```

### Custom DNS Query Testing:
```bash
# DNS with specific query types
nmap --script dns-brute --script-args dns-brute.domain=target.com,dns-brute.type=AAAA -p 53 <target-ip>

# DNS PTR record enumeration (reverse DNS)
nmap --script dns-brute --script-args dns-brute.domain=1.168.192.in-addr.arpa,dns-brute.type=PTR -p 53 <target-ip>

# DNS TXT record enumeration
nmap --script dns-brute --script-args dns-brute.domain=target.com,dns-brute.type=TXT -p 53 <target-ip>

# DNS MX record enumeration
nmap --script dns-brute --script-args dns-brute.domain=target.com,dns-brute.type=MX -p 53 <target-ip>
```

---

## 🎯 Complete DNS Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: DNS port discovery
nmap -sU -p 53 <target>

# Step 2: DNS service version detection
nmap -sU -sV -p 53 <target>

# Step 3: Basic DNS functionality testing
nmap --script dns-recursion,dns-nsid -p 53 <target>
```

### Phase 2: Zone Transfer and Information Gathering
```bash
# Step 4: Zone transfer attempts
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<target_domain> -p 53 <target>

# Step 5: DNS brute force enumeration
nmap --script dns-brute --script-args dns-brute.domain=<target_domain> -p 53 <target>

# Step 6: Service record discovery
nmap --script dns-service-discovery --script-args dns-service-discovery.domain=<target_domain> -p 53 <target>
```

### Phase 3: Security Assessment
```bash
# Step 7: DNS cache snooping
nmap --script dns-cache-snoop -p 53 <target>

# Step 8: DNS update testing
nmap --script dns-update -p 53 <target>

# Step 9: DNS server vulnerability assessment
nmap --script dns-fuzz,dns-random-txid -p 53 <target>
```

---

## 🔁 All-in-One Scan

### Basic DNS Discovery (no authentication)
```bash
nmap -sU -sV -p 53 --script "dns-nsid,dns-recursion,dns-service-discovery" --script-args dns-service-discovery.domain=<target_domain> <target-ip>
```

### Zone Transfer Focused Assessment
```bash
nmap -sT -p 53 --script "dns-zone-transfer" --script-args dns-zone-transfer.domain=<target_domain>,dns-zone-transfer.addall=true <target-ip>
```

### Comprehensive DNS Enumeration
```bash
nmap -sU -sT --script dns-* --script-args dns-zone-transfer.domain=<target_domain>,dns-brute.domain=<target_domain> -p 53 <target-ip>
```

### DNS Security Assessment
```bash
nmap -sU -p 53 --script "dns-recursion,dns-update,dns-cache-snoop,dns-random-txid,dns-random-srcport" <target-ip>
```

---

## 📊 Real Lab Examples

### Example 1: DNS Zone Transfer Discovery
```bash
# Target: DNS server with potential zone transfer vulnerability

# Basic DNS service detection
nmap -sU -sV -p 53 192.168.1.10

# Zone transfer attempt
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=internal.company.com -p 53 192.168.1.10

# Multiple domain zone transfer testing
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=company.com,dns-zone-transfer.addall=true -p 53 192.168.1.10
```

**Expected Output (if vulnerable):**
```
PORT   STATE         SERVICE VERSION
53/udp open|filtered domain

| dns-zone-transfer: 
| internal.company.com:
|   SOA   ns1.internal.company.com. admin.internal.company.com.
|   NS    ns1.internal.company.com.
|   NS    ns2.internal.company.com.
|   A     web.internal.company.com.     192.168.1.100
|   A     mail.internal.company.com.    192.168.1.101
|   A     ftp.internal.company.com.     192.168.1.102
|   A     admin.internal.company.com.   192.168.1.103
|   CNAME intranet.internal.company.com. web.internal.company.com.
|_  MX    mail.internal.company.com.    10
```

### Example 2: DNS Brute Force and Service Discovery
```bash
# Target: DNS server for subdomain enumeration

# DNS brute force with common subdomains
nmap --script dns-brute --script-args dns-brute.domain=company.com -p 53 192.168.1.10

# Service record discovery
nmap --script dns-service-discovery --script-args dns-service-discovery.domain=company.com -p 53 192.168.1.10

# Cache snooping for reconnaissance
nmap --script dns-cache-snoop --script-args dns-cache-snoop.domains={www.company.com,mail.company.com,admin.company.com} -p 53 192.168.1.10
```

### Example 3: DNS Security Assessment
```bash
# Target: DNS server security evaluation

# Recursion testing
nmap --script dns-recursion -p 53 192.168.1.10

# DNS update vulnerability
nmap --script dns-update --script-args dns-update.hostname=malicious.company.com,dns-update.ip=192.168.1.200 -p 53 192.168.1.10

# DNS server information leakage
nmap --script dns-nsid,dns-random-txid -p 53 192.168.1.10
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **No Response** | UDP timeouts or filtered | Use `-Pn` flag, check firewalls |
| **Zone Transfer Refused** | `Transfer failed` | Try different domains or nameservers |
| **Empty Brute Force** | No subdomains found | Use custom wordlists with `dns-brute.hostlist` |
| **UDP Port Issues** | Inconsistent results | Run multiple times, UDP is unreliable |
| **Permission Denied** | Script execution fails | Run with appropriate privileges for UDP |

### Common Error Messages:
- `REFUSED` → Zone transfer not allowed
- `SERVFAIL` → Server cannot process request
- `NXDOMAIN` → Domain does not exist
- `TIMEOUT` → Network connectivity or firewall issues
- `FORMERR` → Malformed DNS query

### Performance Optimization:
```bash
# Faster DNS scanning
nmap -T4 --script dns-brute --script-args dns-brute.threads=20 -p 53 <target>

# Optimized zone transfer
nmap --script dns-zone-transfer --script-args dns-zone-transfer.timeout=5 -p 53 <target>

# Custom timing for UDP
nmap -sU --script dns-* --max-retries=1 --host-timeout=30s -p 53 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test  
- **DNS enumeration can be detected** through monitoring and logs
- **Follow responsible disclosure** if vulnerabilities are found

### Operational Security (OpSec):
- **DNS queries generate logs** - be aware of detection
- **Zone transfers are highly visible** - logged and monitored
- **Brute force generates many queries** - consider rate limiting
- **Cache snooping can be detected** through timing analysis

### Detection Indicators:
Target systems may detect DNS enumeration through:
- **DNS Query Logs**: Unusual query patterns and volumes
- **Zone Transfer Attempts**: Logged and alerted on most systems
- **Brute Force Detection**: High volume of NXDOMAIN responses
- **Security Monitoring**: DNS security tools and SIEM alerts

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| Zone Transfer Allowed | 🔴 Critical | Disable zone transfers to unauthorized hosts |
| Open DNS Recursion | 🟠 High | Restrict recursion to authorized networks |
| DNS Update Allowed | 🔴 Critical | Disable dynamic updates or restrict access |
| Information Disclosure | 🟡 Medium | Review DNS record exposure |
| Weak DNS Security | 🟡 Medium | Implement DNS security best practices |

---

## 🎯 eJPT Focus Points

### Essential DNS Commands for eJPT:
```bash
# Zone transfer testing
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<target_domain> -p 53 <target>

# DNS subdomain brute force
nmap --script dns-brute --script-args dns-brute.domain=<target_domain> -p 53 <target>

# DNS recursion testing
nmap --script dns-recursion -p 53 <target>

# DNS service discovery
nmap --script dns-service-discovery --script-args dns-service-discovery.domain=<target_domain> -p 53 <target>
```

### Key DNS Assessment Areas for eJPT:
1. **Zone Transfer Testing**: Most critical DNS vulnerability to check
2. **Subdomain Enumeration**: Discover hidden services and applications
3. **DNS Server Information**: Version and configuration details
4. **Security Configuration**: Recursion, updates, and access controls
5. **Service Discovery**: Mail servers, web servers, and other services
6. **Cache Analysis**: Internal network reconnaissance capabilities

### Common eJPT DNS Scenarios:
- **Zone Transfer Success**: Complete domain mapping revealing internal infrastructure
- **Subdomain Discovery**: Finding admin panels, staging environments, development servers
- **Internal Domain Exposure**: Discovering internal.company.com through external DNS
- **Service Enumeration**: Identifying mail servers, web servers, FTP servers via DNS records
- **DNS Server Abuse**: Using misconfigured DNS for reconnaissance and attacks

### Critical DNS Information for Further Testing:
- **Discovered Subdomains**: Targets for web application testing
- **Internal IP Ranges**: Network mapping and scanning targets
- **Mail Servers**: Email security assessment targets
- **Administrative Interfaces**: High-value targets for exploitation
- **Development/Staging**: Often less secured environments

---

## 🔗 Integration with Other Tools

After Nmap DNS enumeration, follow up with:
- **dig/nslookup**: Manual DNS queries and verification
- **dnsrecon**: Advanced DNS reconnaissance and enumeration
- **fierce**: Domain scanner and subdomain brute forcer
- **dnsenum**: Comprehensive DNS enumeration tool
- **sublist3r**: Subdomain enumeration using multiple sources
- **amass**: Advanced subdomain and network mapping
- **gobuster**: DNS subdomain brute forcing
- **massdns**: High-performance DNS stub resolver

### DNS Security Testing Workflow:
```bash
# 1. Nmap DNS discovery and enumeration
nmap --script dns-zone-transfer,dns-brute --script-args dns-zone-transfer.domain=target.com,dns-brute.domain=target.com -p 53 <dns_server>

# 2. Extract discovered hosts and subdomains
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' dns_results.txt > discovered_ips.txt
grep -oE '[a-zA-Z0-9.-]+\.target\.com' dns_results.txt > discovered_subdomains.txt

# 3. Follow up with comprehensive scanning
nmap -sS -sV -sC -iL discovered_ips.txt

# 4. Web application testing on discovered subdomains
for subdomain in $(cat discovered_subdomains.txt); do
    echo "Testing $subdomain"
    nmap --script http-enum -p 80,443 $subdomain
done

# 5. Advanced DNS reconnaissance
dnsrecon -d target.com -t axfr
fierce -dns target.com
```

---

## 📝 Output and Documentation

### Saving DNS Assessment Results:
```bash
# Comprehensive DNS assessment
nmap --script dns-* --script-args dns-zone-transfer.domain=target.com,dns-brute.domain=target.com -oA dns_enumeration -p 53 <target>

# Zone transfer results
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=target.com -oN dns_zone_transfer.txt -p 53 <target>

# DNS brute force results
nmap --script dns-brute --script-args dns-brute.domain=target.com -oX dns_subdomains.xml -p 53 <target>
```

### Results Processing and Analysis:
```bash
# Extract discovered subdomains
grep -E "^\|.*\.target\.com" dns_enumeration.nmap

# Find zone transfer data
grep -A 50 "dns-zone-transfer" dns_enumeration.nmap

# Identify DNS security issues
grep -i "recursion\|update\|cache" dns_enumeration.nmap

# Extract IP addresses for further scanning
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' dns_enumeration.nmap | sort -u > discovered_ips.txt
```

---

## 📖 References and Further Reading

### Official Documentation:
- [Nmap NSE Scripts](https://nmap.org/nsedoc/scripts/) - Complete NSE script reference
- [DNS Protocol RFC 1035](https://tools.ietf.org/html/rfc1035) - DNS specification
- [DNS Zone Transfer RFC 5936](https://tools.ietf.org/html/rfc5936) - Zone transfer specification

### Security Research:
- [DNS Security Extensions (DNSSEC)](https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en) - DNS security framework
- [OWASP DNS Security](https://owasp.org/www-community/attacks/DNS_spoofing) - DNS attack methodologies

### Related Tools:
- [dnsrecon](https://github.com/darkoperator/dnsrecon) - DNS reconnaissance tool
- [fierce](https://github.com/mschwager/fierce) - Domain scanner
- [sublist3r](https://github.com/aboul3la/Sublist3r) - Subdomain enumeration

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Scope**: Complete DNS Service Enumeration Guide
