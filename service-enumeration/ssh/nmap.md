# 🔐 SSH Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate SSH (Secure Shell) services and assess their security configuration for penetration testing and security assessments.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-ssh-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [SSH Service Discovery](#-1-ssh-service-discovery-and-version-detection)
- [SSH Algorithm Enumeration](#-2-ssh-algorithm-and-cipher-enumeration)
- [SSH Host Key Analysis](#-3-ssh-host-key-fingerprinting)
- [Authentication Methods](#-4-ssh-authentication-methods-testing)
- [SSH Security Testing](#-5-ssh-brute-force-attacks)
- [SSH Configuration Analysis](#-6-ssh-server-configuration-analysis)
- [Alternative Ports](#-7-ssh-on-alternative-ports)
- [Advanced Techniques](#-8-advanced-ssh-enumeration)
- [Complete Assessment Workflow](#-complete-ssh-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from SSH services, such as:
- SSH server version and software identification
- Supported encryption algorithms and ciphers
- Authentication methods available
- Host key fingerprints for server identification
- Security configuration weaknesses
- Potential backdoors or misconfigurations
- Alternative port usage
- Brute force attack possibilities

---

## 🗂️ SSH Enumeration Scripts Summary

| # | Script                    | What it Does                             | Auth Required | Ports |
|---|---------------------------|------------------------------------------|---------------|-------|
| 1 | banner                    | Grab SSH service banner                  | ❌            | 22,2222 |
| 2 | ssh-hostkey               | Extract SSH host key fingerprints       | ❌            | 22,2222 |
| 3 | ssh2-enum-algos           | Enumerate supported SSH algorithms       | ❌            | 22,2222 |
| 4 | ssh-auth-methods          | Test SSH authentication methods          | ❌            | 22,2222 |
| 5 | ssh-brute                 | Brute force SSH authentication          | ❌            | 22,2222 |
| 6 | ssh-run                   | Execute commands via SSH                 | ✅            | 22,2222 |
| 7 | ssh-publickey-acceptance  | Test SSH public key acceptance           | ❌            | 22,2222 |

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has SSH service enabled** (port 22 or alternative ports)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick SSH Port Check:
```bash
# Check standard SSH port
nmap -p 22 <target-ip>

# Check common alternative SSH ports
nmap -p 22,2222,2223,8022,22022 <target-ip>
```

### Check if SSH is responsive:
```bash
nmap -sV -p 22 <target-ip>
```

---

## 🔧 SSH Service Enumeration Commands

## 🖥️ 1. SSH Service Discovery and Version Detection
```bash
# Basic SSH service detection
nmap -sV -p 22 <target-ip>

# SSH banner grabbing
nmap --script banner -p 22 <target-ip>

# Comprehensive SSH service information
nmap -sV --script banner,ssh-hostkey -p 22 <target-ip>
```
**INE Lab Example:**
```bash
nmap -sV --script banner -p 22 demo.ine.local
```
**Sample Output:**
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.4

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
**Interpretation:**
- SSH service running on port 22
- OpenSSH version 7.4
- Protocol 2.0 supported (good, protocol 1.0 is deprecated)
- Banner reveals exact software version

**Why is this useful:**
- Version information helps identify known vulnerabilities
- Protocol version indicates security level
- Software identification aids in targeted attacks

---

## 🌐 2. SSH Algorithm and Cipher Enumeration
```bash
# Enumerate supported SSH algorithms
nmap --script ssh2-enum-algos -p 22 <target-ip>

# Check for weak encryption algorithms
nmap --script ssh2-enum-algos --script-args ssh.timeout=5 -p 22 <target-ip>
```
**Purpose:**

This script discovers which cryptographic algorithms the SSH server supports for:
- Key exchange (Diffie-Hellman methods)
- Encryption algorithms (AES, ChaCha20, etc.)
- MAC (Message Authentication Code) algorithms
- Compression methods

**INE Lab Example:**
```bash
nmap --script ssh2-enum-algos -p 22 demo.ine.local
```
**Sample Output:**
```bash
| ssh2-enum-algos: 
|   kex_algorithms: (9)
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group16-sha512
|       diffie-hellman-group14-sha256
|   server_host_key_algorithms: (3)
|       rsa-sha2-512
|       rsa-sha2-256
|       ssh-rsa
|   encryption_algorithms: (6)
|       chacha20-poly1305@openssh.com
|       aes128-ctr
|       aes192-ctr
|       aes256-ctr
|       aes128-gcm@openssh.com
|       aes256-gcm@openssh.com
|   mac_algorithms: (10)
|       umac-64-etm@openssh.com
|       umac-128-etm@openssh.com
|       hmac-sha2-256-etm@openssh.com
|       hmac-sha2-512-etm@openssh.com
|       hmac-sha1-etm@openssh.com
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
```

**Security Analysis:**
- **Strong Algorithms**: curve25519, AES-256, ChaCha20-Poly1305
- **Weak Algorithms**: Look for DES, RC4, MD5, SHA1 (not present here - good!)
- **Key Exchange**: Modern elliptic curve methods preferred
- **Compression**: Optional compression available

**Red Flags to Look For:**
- SSH protocol 1.0 support
- Weak ciphers like DES, 3DES, RC4
- Weak MAC algorithms like MD5
- Deprecated Diffie-Hellman groups

---

## 🔑 3. SSH Host Key Fingerprinting
```bash
# SSH host key collection
nmap --script ssh-hostkey -p 22 <target-ip>

# SSH host key with detailed information
nmap --script ssh-hostkey --script-args ssh_hostkey=full -p 22 <target-ip>
```
**Purpose:**

Host key fingerprinting helps:
- Identify and track SSH servers
- Detect man-in-the-middle attacks
- Verify server authenticity
- Track server changes over time

**INE Lab Example:**
```bash
nmap --script ssh-hostkey -p 22 demo.ine.local
```
**Sample Output:**
```bash
| ssh-hostkey: 
|   2048 aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88:99 (RSA)
|   256 11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff:00 (ECDSA)
|_  256 99:88:77:66:55:44:33:22:11:00:ff:ee:dd:cc:bb:aa (ED25519)
```
**Interpretation:**
- **RSA 2048-bit**: Traditional but secure key
- **ECDSA 256-bit**: Elliptic curve key (efficient)
- **ED25519 256-bit**: Modern, highly secure key type

**Security Implications:**
- Multiple key types provide compatibility
- ED25519 is preferred for security and performance
- Key fingerprints can be used for server verification

---

## 🔐 4. SSH Authentication Methods Testing
```bash
# Enumerate SSH authentication methods
nmap --script ssh-auth-methods -p 22 <target-ip>

# Test for password authentication
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 <target-ip>

# Check authentication for specific user
nmap --script ssh-auth-methods --script-args ssh.user=admin -p 22 <target-ip>
```
**Purpose:**

This script discovers which authentication methods are supported:
- Password authentication
- Public key authentication
- Keyboard-interactive authentication
- GSSAPI authentication

**INE Lab Example:**
```bash
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 demo.ine.local
```
**Sample Output:**
```bash
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|     password
|_    keyboard-interactive
```
**Interpretation:**
- **Public key**: Key-based authentication supported
- **Password**: Password authentication enabled
- **Keyboard-interactive**: Interactive authentication available

**Security Assessment:**
- Password authentication = potential brute force target
- Public key only = more secure configuration
- Multiple methods = flexibility but broader attack surface

---

## 🔓 5. SSH Brute Force Attacks
```bash
# Basic SSH brute force
nmap --script ssh-brute -p 22 <target-ip>

# SSH brute force with custom wordlists
nmap --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt -p 22 <target-ip>

# SSH brute force with timeout and thread control
nmap --script ssh-brute --script-args ssh-brute.timeout=5,brute.threads=3 -p 22 <target-ip>

# First successful credential only
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target-ip>
```
**Purpose:**

SSH brute force testing attempts to discover weak credentials by trying common username/password combinations.

**INE Lab Example:**
```bash
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 demo.ine.local
```
**Sample Output:**
```bash
| ssh-brute: 
|   Accounts: 
|     root:password - Valid credentials
|     admin:admin - Valid credentials
|_    user:123456 - Valid credentials
```

**Important Security Considerations:**
- **Only use with proper authorization**
- **Can cause account lockouts**
- **Generate significant logs**
- **Use brute.firstonly=true** to minimize impact

**Attack Prevention:**
- Disable password authentication
- Use strong passwords
- Implement account lockout policies
- Use fail2ban or similar tools

---

## ⚙️ 6. SSH Server Configuration Analysis
```bash
# SSH server software identification
nmap -sV --script banner -p 22 <target-ip>

# SSH configuration weaknesses
nmap --script ssh2-enum-algos -p 22 <target-ip> | grep -E "(weak|obsolete|deprecated)"

# SSH protocol compliance testing
nmap --script ssh2-enum-algos --script-args ssh.timeout=10 -p 22 <target-ip>
```
**Purpose:**

Configuration analysis helps identify:
- Outdated SSH software versions
- Weak cryptographic configurations
- Security misconfigurations
- Compliance with security standards

**Common Misconfigurations:**
- Root login enabled
- Password authentication on production systems
- Weak encryption algorithms enabled
- SSH protocol 1.0 support

---

## 🚪 7. SSH on Alternative Ports
```bash
# Standard SSH port
nmap --script ssh-* -p 22 <target-ip>

# Common alternative SSH ports
nmap --script ssh-* -p 2222,2223,22022,22222 <target-ip>

# SSH on high ports
nmap --script ssh-hostkey -p 8022,9022,10022 <target-ip>

# Comprehensive SSH port scanning
nmap -sV -p 22,2222,2223,8022,9022,10022,22022,22222 <target-ip>
```
**Purpose:**

Many administrators run SSH on non-standard ports for:
- Security through obscurity
- Avoiding automated attacks
- Running multiple SSH instances
- Compliance requirements

**INE Lab Example:**
```bash
nmap -sV -p 22,2222,8022 demo.ine.local
```
**Sample Output:**
```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
2222/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
8022/tcp open  ssh     Dropbear sshd 2019.78 (protocol 2.0)
```

**Analysis:**
- Multiple SSH services running
- Different SSH implementations (OpenSSH vs Dropbear)
- Version differences may indicate different purposes

---

## 🔧 8. Advanced SSH Enumeration

### SSH Command Execution (When Credentials Available)
```bash
# SSH command execution
nmap --script ssh-run --script-args ssh-run.cmd="id",ssh-run.username=user,ssh-run.password=pass -p 22 <target-ip>

# Multiple commands
nmap --script ssh-run --script-args 'ssh-run.cmd="whoami;pwd;ls -la"',ssh-run.username=admin,ssh-run.password=admin123 -p 22 <target-ip>

# System information gathering
nmap --script ssh-run --script-args ssh-run.cmd="uname -a",ssh-run.username=root,ssh-run.password=toor -p 22 <target-ip>
```

### SSH Public Key Testing
```bash
# SSH public key acceptance testing
nmap --script ssh-publickey-acceptance -p 22 <target-ip>

# Test specific SSH public keys
nmap --script ssh-publickey-acceptance --script-args publickeys=key.pub -p 22 <target-ip>
```

### SSH Honeypot Detection
```bash
# Unusual SSH banner detection
nmap --script banner -p 22 <target-ip> | grep -i "honey\|fake\|trap"

# SSH server behavior analysis
nmap --script ssh-auth-methods --script-args ssh.user=root,ssh.password=root -p 22 <target-ip>

# Response timing analysis
nmap -T4 --script ssh-hostkey -p 22 <target-ip>
```

---

## 🎯 Complete SSH Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: SSH port discovery
nmap -sS -p 22,2222,2223,8022,22022 <target>

# Step 2: SSH service version detection
nmap -sV -p 22 <target>

# Step 3: SSH server identification
nmap --script banner,ssh-hostkey -p 22 <target>
```

### Phase 2: Configuration Analysis
```bash
# Step 4: SSH algorithm enumeration
nmap --script ssh2-enum-algos -p 22 <target>

# Step 5: Authentication methods testing
nmap --script ssh-auth-methods -p 22 <target>

# Step 6: Host key fingerprinting
nmap --script ssh-hostkey -p 22 <target>
```

### Phase 3: Security Testing
```bash
# Step 7: SSH brute force (if appropriate and authorized)
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target>

# Step 8: SSH configuration weakness assessment
nmap --script ssh2-enum-algos -p 22 <target> | grep -E "(des|md5|rc4|sha1)"

# Step 9: Alternative port scanning
nmap --script ssh-* -p 2222,8022,22022 <target>
```

---

## 🔁 All-in-One Scan

### Basic SSH Discovery (no authentication)
```bash
nmap -sV -p 22,2222,8022 --script "ssh-hostkey,ssh2-enum-algos,ssh-auth-methods,banner" <target-ip>
```

### Security Assessment Scan
```bash
nmap -p 22,2222,8022 --script "ssh2-enum-algos,ssh-auth-methods,ssh-hostkey" <target-ip>
```

### Comprehensive SSH Assessment
```bash
nmap --script ssh-* -p 22,2222,2223,8022,22022 <target-ip>
```

### SSH Brute Force (Use with Extreme Caution)
```bash
nmap --script ssh-brute --script-args brute.firstonly=true,brute.delay=2 -p 22 <target-ip>
```

---

## 📊 Real Lab Examples

### Example 1: Standard SSH Service Assessment
```bash
# Target: Standard SSH server on port 22

# Basic service detection
nmap -sV -p 22 192.168.1.100

# SSH configuration analysis
nmap --script ssh2-enum-algos,ssh-hostkey -p 22 192.168.1.100

# Authentication methods testing
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 192.168.1.100
```

**Expected Output:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)

| ssh-hostkey: 
|   2048 aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88:99 (RSA)
|   256 11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff:00 (ECDSA)
|_  256 99:88:77:66:55:44:33:22:11:00:ff:ee:dd:cc:bb:aa (ED25519)

| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|_    password
```

### Example 2: SSH Security Weakness Detection
```bash
# Target: SSH server with potential security issues

# Algorithm analysis for weak ciphers
nmap --script ssh2-enum-algos -p 22 192.168.1.200 | grep -E "(des|md5|rc4|sha1)"

# Root login testing
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 192.168.1.200

# Brute force testing (authorized only)
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 192.168.1.200
```

### Example 3: Multiple SSH Services Assessment
```bash
# Target: Server with SSH on multiple ports

# Port discovery
nmap -sS -p 22,2222,8022,22022 192.168.1.50

# Service comparison
nmap -sV --script ssh-hostkey -p 22,2222,8022 192.168.1.50

# Configuration differences
nmap --script ssh2-enum-algos -p 22,2222 192.168.1.50
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Refused** | `Connection refused` | Check if SSH service is running |
| **Connection Timeout** | Scripts hang or timeout | Use `--script-args ssh.timeout=10s` |
| **Authentication Failed** | Login failures | Verify credentials and account status |
| **No Response** | Empty results | Check firewall rules and network connectivity |
| **Slow Scanning** | Long execution times | Use `-T4` for faster scanning |

### Common Error Messages:
- `ssh: connect to host port 22: Connection refused` → SSH service not running
- `ssh: connect to host port 22: No route to host` → Network connectivity issue
- `Permission denied (publickey)` → Key-based authentication only
- `Permission denied (publickey,password)` → Invalid credentials
- `Connection reset by peer` → Potential security system blocking

### Performance Optimization:
```bash
# Faster SSH scanning
nmap -T4 --script ssh-hostkey,ssh2-enum-algos -p 22 <target>

# Slower scanning to avoid detection
nmap -T2 --script ssh-* -p 22 <target>

# Parallel SSH assessment with delays
nmap --script ssh-brute --script-args brute.threads=2,brute.delay=3 -p 22 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **SSH brute force can cause account lockouts** - use sparingly
- **Follow responsible disclosure** if vulnerabilities are found

### Operational Security (OpSec):
- **SSH enumeration generates logs** - be aware of detection
- **Brute force attacks are highly visible** - use with extreme caution
- **Failed login attempts trigger alerts** - consider detection mechanisms
- **Timing attacks can be detected** - use appropriate delays

### Detection Indicators:
Target systems may detect SSH enumeration through:
- **SSH Logs**: Connection attempts and authentication failures
- **Network Monitoring**: Unusual SSH traffic patterns
- **Failed Login Alerts**: Multiple authentication failures
- **Intrusion Detection**: SSH scanning signatures

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| SSH Protocol 1.0 Support | 🔴 Critical | Disable immediately |
| Root Password Login | 🔴 Critical | Disable root password access |
| Weak Encryption Algorithms | 🟠 High | Update cipher configuration |
| Default Credentials | 🔴 Critical | Change all default passwords |
| SSH on Standard Port | 🟡 Medium | Consider port changes for security |

---

## 🎯 eJPT Focus Points

### Essential SSH Commands for eJPT:
```bash
# Basic SSH discovery and version detection
nmap -sV --script ssh-hostkey -p 22 <target>

# SSH authentication methods
nmap --script ssh-auth-methods -p 22 <target>

# SSH brute force (when appropriate)
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target>

# SSH algorithm enumeration
nmap --script ssh2-enum-algos -p 22 <target>
```

### Key SSH Assessment Areas for eJPT:
1. **Service Identification**: Version detection and banner grabbing
2. **Authentication Testing**: Available authentication methods and brute force
3. **Algorithm Analysis**: Supported encryption and key exchange algorithms
4. **Host Key Fingerprinting**: SSH server identification and tracking
5. **Alternative Ports**: SSH services on non-standard ports
6. **Security Configuration**: Root login, password authentication settings

### Common eJPT SSH Scenarios:
- **Weak Credentials**: SSH brute force leading to system access
- **SSH Tunneling**: Port forwarding for lateral movement
- **Key-Based Authentication**: SSH key discovery and usage
- **Alternative Ports**: SSH services on non-standard ports for persistence
- **Configuration Analysis**: Identifying SSH misconfigurations

### Critical SSH Information for Further Testing:
- **Valid Credentials**: Username/password combinations for system access
- **SSH Server Version**: Software information for vulnerability research
- **Supported Algorithms**: Cryptographic strength assessment
- **Host Key Fingerprints**: Server identification and man-in-the-middle detection
- **Authentication Methods**: Available login mechanisms

---

## 🔗 Integration with Other Tools

After Nmap SSH enumeration, follow up with:
- **Hydra**: Advanced SSH brute force attacks
- **ssh/sshpass**: Manual SSH client access
- **ssh-keygen**: SSH key generation and analysis
- **Metasploit**: SSH auxiliary modules and exploits
- **paramiko**: Python SSH library for custom scripts
- **ssh-audit**: Comprehensive SSH configuration auditing
- **John the Ripper**: SSH key cracking

### SSH Security Testing Workflow:
```bash
# 1. Nmap SSH discovery and enumeration
nmap --script ssh-* -p 22,2222,8022 <target>

# 2. Extract weak algorithms for further investigation
grep -E "(des|md5|rc4|sha1)" ssh_results.txt

# 3. Advanced SSH configuration auditing
ssh-audit <target>

# 4. Manual SSH testing (with valid credentials)
ssh -v user@<target>

# 5. SSH tunneling and pivoting (post-access)
ssh -L 8080:internal-server:80 user@<target>
```

---

## 📝 Output and Documentation

### Saving SSH Assessment Results:
```bash
# Comprehensive SSH assessment
nmap --script ssh-* -oA ssh_enumeration -p 22,2222,8022 <target>

# SSH algorithm analysis
nmap --script ssh2-enum-algos -oN ssh_algorithms.txt -p 22 <target>

# SSH brute force results (if authorized)
nmap --script ssh-brute -oX ssh_bruteforce.xml -p 22 <target>
```

### Results Processing and Analysis:
```bash
# Extract SSH host keys
grep -A 10 "ssh-hostkey" ssh_enumeration.nmap

# Find weak SSH algorithms
grep -E "(des|md5|rc4|sha1)" ssh_enumeration.nmap

# Identify successful SSH credentials
grep -A 5 "Valid credentials" ssh_enumeration.nmap

# Extract SSH version information
grep "ssh.*OpenSSH\|ssh.*Dropbear" ssh_enumeration.nmap
```

---

## 📖 References and Further Reading

### Official Documentation:
- [Nmap NSE Scripts](https://nmap.org/nsedoc/scripts/) - Complete NSE script reference
- [OpenSSH Documentation](https://www.openssh.com/) - Official OpenSSH documentation
- [SSH Protocol Specifications](https://tools.ietf.org/html/rfc4253) - SSH protocol RFC

### Security Research:
- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/security) - SSH security guidelines
- [SSH Hardening Guide](https://linux-audit.com/audit-and-harden-your-ssh-configuration/) - SSH configuration hardening

### Related Tools:
- [ssh-audit](https://github.com/jtesta/ssh-audit) - SSH configuration auditing
- [Hydra](https://github.com/vanhauser-thc/thc-hydra) - Network login cracker
- [Metasploit](https://www.metasploit.com/) - Penetration testing framework

---

**Last Updated**: September 2025  
**Version**: 2.0  
**Scope**: Complete SSH Service Enumeration Guide
