# 🔐 SSH Service Enumeration with Nmap

Complete guide for SSH (Secure Shell) service discovery and enumeration using Nmap NSE scripts.

---

## 🔧 Basic SSH Enumeration Commands

### Essential SSH Scripts
```bash
# SSH version and algorithm enumeration
nmap --script ssh2-enum-algos -p 22 <target>

# SSH host key fingerprinting
nmap --script ssh-hostkey -p 22 <target>

# SSH authentication methods detection
nmap --script ssh-auth-methods -p 22 <target>

# All SSH scripts (comprehensive)
nmap --script ssh-* -p 22 <target>
```

## 🔍 SSH Service Discovery and Analysis

### Version and Protocol Detection
```bash
# SSH service version detection
nmap -sV -p 22 <target>

# SSH protocol version support (SSH1/SSH2)
nmap --script ssh2-enum-algos -p 22 <target>

# SSH server banner and version information
nmap --script banner -p 22 <target>

# Comprehensive SSH service information
nmap -sV --script ssh-hostkey,ssh2-enum-algos -p 22 <target>
```

### Algorithm and Cipher Enumeration
```bash
# Enumerate supported SSH algorithms
nmap --script ssh2-enum-algos -p 22 <target>

# Check for weak encryption algorithms
nmap --script ssh2-enum-algos --script-args ssh.timeout=5 -p 22 <target>

# SSH key exchange algorithms
nmap --script ssh2-enum-algos -p 22 <target> | grep "kex_algorithms"

# SSH cipher and MAC algorithms
nmap --script ssh2-enum-algos -p 22 <target> | grep -E "(encryption_algorithms|mac_algorithms)"
```

## 🛡️ SSH Security Assessment

### Authentication Method Testing
```bash
# Enumerate SSH authentication methods
nmap --script ssh-auth-methods -p 22 <target>

# Test for password authentication
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 <target>

# Check for key-based authentication
nmap --script ssh-auth-methods --script-args ssh.user=admin -p 22 <target>

# Multiple user authentication testing
nmap --script ssh-auth-methods --script-args ssh.user=user,ssh.password=password -p 22 <target>
```

### SSH Brute Force Attacks
```bash
# Basic SSH brute force
nmap --script ssh-brute -p 22 <target>

# SSH brute force with custom wordlists
nmap --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt -p 22 <target>

# SSH brute force with timeout and thread control
nmap --script ssh-brute --script-args ssh-brute.timeout=5,brute.threads=3 -p 22 <target>

# First successful credential only
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target>
```

## 🔑 SSH Key and Certificate Analysis

### Host Key Fingerprinting
```bash
# SSH host key collection
nmap --script ssh-hostkey -p 22 <target>

# SSH host key with detailed information
nmap --script ssh-hostkey --script-args ssh_hostkey=full -p 22 <target>

# Multiple SSH host key formats
nmap --script ssh-hostkey --script-args ssh_hostkey='visual bubble' -p 22 <target>

# Compare SSH host keys across multiple targets
nmap --script ssh-hostkey -p 22 <target_range>
```

### Public Key Enumeration
```bash
# SSH public key acceptance testing
nmap --script ssh-publickey-acceptance -p 22 <target>

# Test specific SSH public keys
nmap --script ssh-publickey-acceptance --script-args publickeys=key.pub -p 22 <target>

# SSH certificate validation
nmap --script ssl-cert -p 22 <target> # For SSH over SSL tunnels
```

## 🚪 SSH Access and Backdoor Detection

### SSH Backdoor Detection
```bash
# Generic SSH backdoor detection
nmap --script ssh-run --script-args ssh-run.cmd="id" -p 22 <target>

# SSH command execution (if credentials available)
nmap --script ssh-run --script-args ssh-run.cmd="uname -a",ssh-run.username=user,ssh-run.password=pass -p 22 <target>

# SSH backdoor command testing
nmap --script ssh-run --script-args 'ssh-run.cmd="whoami;pwd;ls -la"' -p 22 <target>
```

### Unusual SSH Configurations
```bash
# Check for SSH on non-standard ports
nmap --script ssh-* -p 2222,2223,22022,22222 <target>

# SSH service on unusual ports
nmap -sV -p 1-65535 <target> | grep ssh

# Multiple SSH service instances
nmap --script ssh-hostkey -p 22,2222,22022 <target>
```

## 🔧 Advanced SSH Enumeration

### SSH Server Configuration Analysis
```bash
# SSH server software identification
nmap -sV --script banner -p 22 <target>

# SSH configuration weaknesses
nmap --script ssh2-enum-algos -p 22 <target> | grep -E "(weak|obsolete|deprecated)"

# SSH protocol compliance testing
nmap --script ssh2-enum-algos --script-args ssh.timeout=10 -p 22 <target>
```

### Performance and Timing Analysis
```bash
# SSH response time analysis
nmap --script ssh-auth-methods --script-args ssh.timeout=3 -p 22 <target>

# SSH connection testing with timing
nmap -T4 --script ssh-hostkey,ssh2-enum-algos -p 22 <target>

# SSH service availability testing
nmap -Pn --script ssh-* -p 22 <target>
```

## 📊 SSH Port Variations and Services

### Standard and Alternative SSH Ports
```bash
# Standard SSH port
nmap --script ssh-* -p 22 <target>

# Common alternative SSH ports
nmap --script ssh-* -p 2222,2223,22022,22222 <target>

# SSH on high ports
nmap --script ssh-hostkey -p 8022,9022,10022 <target>

# Comprehensive SSH port scanning
nmap -sV -p 22,2222,2223,8022,9022,10022,22022,22222 <target>
```

### SSH-Related Services
```bash
# SFTP (SSH File Transfer Protocol)
nmap -sV -p 22 <target>

# SCP (Secure Copy Protocol)
nmap --script ssh-* -p 22 <target>

# SSH tunneling detection
nmap --script ssh-run --script-args ssh-run.cmd="netstat -an" -p 22 <target>
```

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
# Step 7: SSH brute force (if necessary)
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target>

# Step 8: SSH backdoor detection
nmap --script ssh-run --script-args ssh-run.cmd="id" -p 22 <target>

# Step 9: SSH vulnerability assessment
nmap --script ssh-* -p 22 <target>
```

## 📋 Real Lab Examples

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

| ssh2-enum-algos: 
|   kex_algorithms: (9)
|       curve25519-sha256@libssh.org
|       ecdh-sha2-nistp256
|       ecdh-sha2-nistp384
|       ecdh-sha2-nistp521
|       diffie-hellman-group-exchange-sha256
|       diffie-hellman-group16-sha512
|       diffie-hellman-group18-sha512
|       diffie-hellman-group14-sha256
|       diffie-hellman-group14-sha1
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
|       umac-64@openssh.com
|       umac-128@openssh.com
|       hmac-sha2-256
|       hmac-sha2-512
|       hmac-sha1
|   compression_algorithms: (2)
|       none
|_      zlib@openssh.com
```

### Example 2: SSH Brute Force Attack
```bash
# Target: SSH server with weak credentials

# Authentication methods check
nmap --script ssh-auth-methods --script-args ssh.user=admin -p 22 192.168.1.200

# Brute force attack
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 192.168.1.200

# Custom wordlist brute force
nmap --script ssh-brute --script-args userdb=common_users.txt,passdb=common_passwords.txt -p 22 192.168.1.200
```

### Example 3: SSH on Alternative Ports
```bash
# Target: SSH services on non-standard ports

# Port discovery
nmap -sS -p 2222,8022,22022 192.168.1.50

# Service identification
nmap -sV -p 2222,8022,22022 192.168.1.50

# SSH enumeration on alternative ports
nmap --script ssh-* -p 2222 192.168.1.50
```

## ⚠️ SSH Security Issues and Vulnerabilities

### Common SSH Misconfigurations
```bash
# Check for weak SSH algorithms
nmap --script ssh2-enum-algos -p 22 <target> | grep -E "(des|md5|sha1|rc4)"

# Root login detection
nmap --script ssh-auth-methods --script-args ssh.user=root -p 22 <target>

# Password authentication enabled
nmap --script ssh-auth-methods -p 22 <target> | grep "password"

# SSH protocol version 1 support (deprecated)
nmap --script ssh2-enum-algos -p 22 <target> | grep "protocol 1"
```

### SSH Attack Vectors
```bash
# 1. Weak credentials
nmap --script ssh-brute --script-args brute.firstonly=true -p 22 <target>

# 2. SSH user enumeration
nmap --script ssh-auth-methods --script-args ssh.user=nonexistent -p 22 <target>

# 3. SSH key-based attacks
nmap --script ssh-publickey-acceptance -p 22 <target>

# 4. SSH backdoor detection
nmap --script ssh-run --script-args ssh-run.cmd="ps aux" -p 22 <target>
```

## 🛠️ Advanced SSH Analysis Techniques

### SSH Traffic Analysis
```bash
# SSH connection behavior analysis
nmap --script ssh-auth-methods --script-args ssh.timeout=1 -p 22 <target>

# SSH server response timing
nmap -T4 --script ssh-hostkey -p 22 <target>

# SSH service availability
nmap -Pn --script ssh-auth-methods -p 22 <target>
```

### SSH Honeypot Detection
```bash
# Unusual SSH banner detection
nmap --script banner -p 22 <target> | grep -i "honey\|fake\|trap"

# SSH server behavior analysis
nmap --script ssh-auth-methods --script-args ssh.user=root,ssh.password=root -p 22 <target>

# SSH response pattern analysis
nmap --script ssh2-enum-algos,ssh-hostkey -p 22 <target>
```

## 📝 SSH Enumeration Output and Analysis

### Saving SSH Assessment Results
```bash
# Comprehensive SSH assessment
nmap --script ssh-* -oA ssh_enumeration -p 22 <target>

# SSH algorithm analysis
nmap --script ssh2-enum-algos -oN ssh_algorithms.txt -p 22 <target>

# SSH brute force results
nmap --script ssh-brute -oX ssh_bruteforce.xml -p 22 <target>
```

### Results Processing and Analysis
```bash
# Extract SSH host keys
grep -A 10 "ssh-hostkey" ssh_enumeration.nmap

# Find weak SSH algorithms
grep -E "(des|md5|rc4|sha1)" ssh_enumeration.nmap

# Identify successful SSH credentials
grep -A 5 "Valid credentials" ssh_enumeration.nmap
```

## 🎯 eJPT Focus Points

### Essential SSH Commands for eJPT
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
1. **Service Identification:** Version detection and banner grabbing
2. **Authentication Testing:** Available authentication methods and brute force
3. **Algorithm Analysis:** Supported encryption and key exchange algorithms
4. **Host Key Fingerprinting:** SSH server identification and tracking
5. **Alternative Ports:** SSH services on non-standard ports
6. **Security Configuration:** Root login, password authentication settings

### Common eJPT SSH Scenarios:
- **Weak Credentials:** SSH brute force leading to system access
- **SSH Tunneling:** Port forwarding for lateral movement
- **Key-Based Authentication:** SSH key discovery and usage
- **Alternative Ports:** SSH services on non-standard ports for persistence
- **Configuration Analysis:** Identifying SSH misconfigurations

## 🔗 Integration with Other Tools

After Nmap SSH enumeration, follow up with:
- **Hydra:** Advanced SSH brute force attacks
- **ssh/sshpass:** Manual SSH client access
- **ssh-keygen:** SSH key generation and analysis
- **Metasploit:** SSH auxiliary modules and exploits
- **paramiko:** Python SSH library for custom scripts
- **ssh-audit:** Comprehensive SSH configuration auditing

This comprehensive guide covers all essential SSH enumeration techniques for penetration testing and certification preparation, focusing on practical scenarios commonly encountered in eJPT and other security assessments.
