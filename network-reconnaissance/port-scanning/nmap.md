# 🚪 Nmap Guide - Port Scanning

Nmap is the most powerful tool for **port scanning** in penetration testing and network security assessments. This guide focuses exclusively on mastering port scanning techniques.

---

## 🔎 Core Port Scanning Methods

### TCP Scanning Techniques

#### **TCP Connect Scan (-sT)**
- **Description**: Completes the full TCP 3-way handshake
- **Privileges**: Works without root privileges
- **Stealth**: Easy to detect by firewalls/IDS
- **Speed**: Slower due to full connection establishment
- **Use Case**: When root access is not available

```bash
# Basic TCP Connect scan
nmap -sT <target-ip>

# TCP Connect on specific ports
nmap -sT -p 80,443 <target-ip>
```

#### **SYN Scan (-sS) - "Stealth Scan"**
- **Description**: Half-open scan (does not complete handshake)
- **Privileges**: Requires root privileges
- **Stealth**: Faster and more stealthy
- **Speed**: Much faster than TCP Connect
- **Use Case**: Default choice for penetration testing

```bash
# Basic SYN scan (requires root)
sudo nmap -sS <target-ip>

# SYN scan with timing optimization
sudo nmap -sS -T4 <target-ip>
```

#### **TCP Null/FIN/Xmas Scans**
Advanced stealth scanning techniques:

```bash
# NULL scan - sends packet with no flags
nmap -sN <target-ip>

# FIN scan - sends packet with FIN flag
nmap -sF <target-ip>

# Xmas scan - sends packet with FIN, PSH, and URG flags
nmap -sX <target-ip>
```

### UDP Port Scanning

#### **UDP Scan (-sU)**
- **Description**: Scans UDP ports for connectionless services
- **Speed**: Very slow compared to TCP scans
- **Challenge**: Many false negatives due to UDP nature
- **Services**: DNS (53), SNMP (161), DHCP (67/68), TFTP (69)

```bash
# Basic UDP scan
sudo nmap -sU <target-ip>

# UDP scan on common ports
sudo nmap -sU -p 53,161,162,69,123 <target-ip>

# Fast UDP scan (top ports only)
sudo nmap -sU --top-ports 100 <target-ip>

# Combined TCP SYN + UDP scan
sudo nmap -sS -sU -p- <target-ip>
```

### 📊 Scan Type Comparison

| Scan Type | Flag | Speed | Stealth | Root Required | Best For |
|-----------|------|-------|---------|---------------|-----------|
| **TCP Connect** | `-sT` | Slow | Low | No | Limited privilege environments |
| **SYN Scan** | `-sS` | Fast | High | Yes | General port scanning |
| **UDP Scan** | `-sU` | Very Slow | Medium | Yes | UDP service discovery |
| **NULL Scan** | `-sN` | Medium | Very High | Yes | Firewall evasion |
| **FIN Scan** | `-sF` | Medium | Very High | Yes | IDS evasion |
| **Xmas Scan** | `-sX` | Medium | Very High | Yes | Advanced stealth |

---

## 🔧 Port Specification Techniques

### Basic Port Selection
```bash
# Scan single port
nmap -p 80 <target-ip>

# Scan multiple specific ports
nmap -p 21,22,80,443 <target-ip>

# Scan port range
nmap -p 1-1000 <target-ip>

# Scan all 65535 ports
nmap -p- <target-ip>
```

### Advanced Port Selection
```bash
# Scan top 100 most common ports
nmap --top-ports 100 <target-ip>

# Scan top 1000 ports (default behavior)
nmap --top-ports 1000 <target-ip>

# Exclude specific ports
nmap --exclude-ports 80,443 <target-ip>

# TCP and UDP ports together
nmap -p T:80,443,U:53,161 <target-ip>
```

### Protocol-Specific Port Scanning
```bash
# TCP ports only
nmap -p T:1-65535 <target-ip>

# UDP ports only  
nmap -p U:1-65535 <target-ip>

# Mixed TCP and UDP
nmap -p T:80,443,U:53,161,162 <target-ip>
```

---

## 🎯 Common Service Port Ranges

### Web Services
```bash
# Standard web ports
nmap -p 80,443 <target-ip>

# Extended web services
nmap -p 80,443,8080,8443,8000,8888,9000,9443 <target-ip>

# Alternative HTTP ports
nmap -p 81,82,83,8001,8002,8008,8080,8081,8082,8083 <target-ip>
```

### Windows Services
```bash
# Essential Windows ports
nmap -p 135,139,445 <target-ip>

# Windows remote access
nmap -p 3389,5985,5986 <target-ip>

# Complete Windows assessment
nmap -p 135,139,445,3389,5985,5986,1433,1434 <target-ip>
```

### Database Services
```bash
# Common database ports
nmap -p 1433,3306,5432,1521,27017 <target-ip>

# Extended database scanning
nmap -p 1433,1434,3306,5432,1521,1522,27017,27018,6379 <target-ip>
```

### Network Infrastructure
```bash
# Network services
nmap -p 53,67,68,161,162,514,123 <target-ip>

# Network management
nmap -p 161,162,514,199,830 <target-ip>
```

### Mail Services  
```bash
# Email ports
nmap -p 25,110,143,993,995,587 <target-ip>

# Extended mail services
nmap -p 25,110,143,993,995,587,465,2525 <target-ip>
```

### Remote Access Services
```bash
# SSH and Telnet
nmap -p 22,23 <target-ip>

# VNC services
nmap -p 5900,5901,5902 <target-ip>

# Remote desktop protocols
nmap -p 3389,1494,2598 <target-ip>
```

---

## 🚀 Performance Optimization

### Timing Templates
Control scan speed and detection avoidance:

| Template | Flag | Speed | Stealth | Use Case |
|----------|------|-------|---------|----------|
| **Paranoid** | `-T0` | Extremely Slow | Maximum | IDS avoidance |
| **Sneaky** | `-T1` | Very Slow | High | Stealth required |
| **Polite** | `-T2` | Slow | Medium | Production systems |
| **Normal** | `-T3` | Normal | Medium | General purpose |
| **Aggressive** | `-T4` | Fast | Low | Lab environments |
| **Insane** | `-T5` | Very Fast | None | Speed over accuracy |

### Timing Examples
```bash
# Maximum stealth (very slow)
nmap -T0 -sS -p 80,443 <target-ip>

# Production-safe scanning
nmap -T2 -sS --top-ports 1000 <target-ip>

# Fast lab scanning
nmap -T4 -sS -p- <target-ip>

# Speed over everything (may miss ports)
nmap -T5 -sS -F <target-ip>
```

### Advanced Performance Tuning
```bash
# Control parallel probes
nmap --min-parallelism 10 --max-parallelism 50 <target-ip>

# Control send rate
nmap --min-rate 100 --max-rate 1000 <target-ip>

# Optimize retries
nmap --max-retries 2 <target-ip>

# Control timeout
nmap --host-timeout 5m <target-ip>

# Skip DNS resolution for speed
nmap -n <target-ip>
```

---

## 🛡️ Host Discovery vs Port Scanning

### Host Discovery Methods
```bash
# Ping sweep (ICMP discovery)
nmap -sn 192.168.1.0/24

# TCP SYN discovery
nmap -sn -PS80,443 192.168.1.0/24

# UDP discovery
nmap -sn -PU53 192.168.1.0/24

# Skip host discovery (assume host is up)
nmap -Pn <target-ip>

# ARP discovery (local network only)
nmap -sn -PR 192.168.1.0/24
```

### When to Skip Host Discovery
```bash
# When ping is blocked
nmap -Pn -sS -p- <target-ip>

# For comprehensive scanning
nmap -Pn -sS -sU --top-ports 1000 <target-ip>

# When targeting specific services
nmap -Pn -p 80,443,22,21 <target-ip>
```

---

## 💾 Port Scanning Output & Documentation

### Output Formats
```bash
# Normal text output
nmap -sS -p- -oN portscan.txt <target-ip>

# XML output (for tools integration)
nmap -sS -p- -oX portscan.xml <target-ip>

# Grepable format
nmap -sS -p- -oG portscan.gnmap <target-ip>

# All formats at once
nmap -sS -p- -oA comprehensive_portscan <target-ip>
```

### Interpreting Port States
```bash
# Port states in Nmap output:
# open      - Service is listening
# closed    - Port is accessible but no service
# filtered  - Port is blocked by firewall
# unfiltered - Port is accessible but state unknown
# open|filtered - Can't determine if open or filtered
# closed|filtered - Can't determine if closed or filtered
```

---

## 🧪 Practical Port Scanning Workflows

### Quick Assessment Workflow
```bash
# Step 1: Fast initial scan
nmap -T4 -F <target-ip>

# Step 2: Scan discovered services
nmap -T4 -p <discovered-ports> <target-ip>

# Step 3: Full port scan if needed
nmap -T4 -p- <target-ip>
```

### Comprehensive Assessment
```bash
# Phase 1: Network discovery
nmap -sn 192.168.1.0/24

# Phase 2: Quick port scan on live hosts
nmap -T4 -F --open <target-list>

# Phase 3: Full TCP port scan
nmap -sS -T4 -p- --open -oA tcp_scan <target-ip>

# Phase 4: UDP port scan (selective)
nmap -sU -T4 --top-ports 100 --open -oA udp_scan <target-ip>
```

### Stealth Assessment Workflow
```bash
# Ultra-stealth approach
nmap -T1 -sS -f --data-length 25 -p 80,443 <target-ip>

# Multiple stealth techniques combined
nmap -T1 -sS -f -D RND:10 --source-port 53 -p- <target-ip>

# Null scan for firewall evasion
nmap -T2 -sN -p- <target-ip>
```

---

## 🔍 Advanced Port Scanning Techniques

### Firewall Evasion
```bash
# Fragment packets
nmap -f -sS <target-ip>

# Use MTU fragmentation
nmap --mtu 8 -sS <target-ip>

# Decoy scanning
nmap -D 192.168.1.100,192.168.1.101,ME -sS <target-ip>

# Random decoys
nmap -D RND:10 -sS <target-ip>

# Source port spoofing
nmap --source-port 53 -sS <target-ip>

# Idle scan (requires zombie host)
nmap -sI zombie-ip:probe-port <target-ip>
```

### IDS Evasion Techniques
```bash
# Very slow timing with randomization
nmap -T0 --randomize-hosts -sS <target-range>

# Mixed scan types
nmap -sS -sF -sN --randomize-hosts <target-ip>

# Custom data payloads
nmap --data-length 200 -sS <target-ip>

# Bad checksum (some systems ignore)
nmap --badsum -sS <target-ip>
```

### IPv6 Port Scanning
```bash
# IPv6 address scanning
nmap -6 -sS <ipv6-address>

# IPv6 network discovery
nmap -6 -sn 2001:db8::/32

# IPv6 port scan with timing
nmap -6 -T4 -sS -p- <ipv6-address>
```

---

## 📊 Port Scanning Best Practices

### Scanning Strategy
1. **Start Fast, Go Deep**: Begin with `-F` or `--top-ports`, then expand
2. **Document Everything**: Always use `-oA` for complete records
3. **Consider Impact**: Use appropriate timing for the environment
4. **Verify Results**: Re-scan interesting ports with different methods
5. **Combine Techniques**: Mix TCP SYN and UDP scanning

### Common Scanning Patterns
```bash
# Discovery → Quick → Detailed
nmap -sn <network>                    # Discovery
nmap -T4 -F <live-hosts>             # Quick assessment  
nmap -sS -T4 -p- <interesting-hosts>  # Detailed scanning

# Stealth progression
nmap -sS -T2 --top-ports 100 <target>  # Initial stealth
nmap -sN -T1 -p- <target>              # Deep stealth
nmap -sF -T1 <open-ports> <target>     # Verification
```

### Error Handling & Troubleshooting
```bash
# When getting "Permission denied"
sudo nmap -sS <target-ip>

# When ports appear closed but shouldn't be
nmap -sT <target-ip>  # Try TCP Connect
nmap -Pn <target-ip>  # Skip ping check

# When scans are too slow
nmap -T4 --min-rate 1000 <target-ip>

# When getting filtered results
nmap -sN <target-ip>  # Try NULL scan
nmap --source-port 53 <target-ip>  # Try source port spoofing
```

---

## 🎯 eJPT Port Scanning Focus

### Essential Commands for eJPT
```bash
# Basic discovery and quick assessment
nmap -sn <network-range>
nmap -sS -T4 -F <target-ip>

# Comprehensive port scanning
nmap -sS -T4 -p- --open <target-ip>
nmap -sU -T4 --top-ports 100 <target-ip>

# Documentation (crucial for reporting)
nmap -sS -T4 -p- -oA portscan_results <target-ip>

# Common service ports
nmap -sS -p 21,22,23,25,53,80,110,135,139,143,443,445,993,995,3306,3389,5432 <target-ip>
```

### Key Port Scanning Skills for eJPT:
1. **TCP SYN Scanning**: Master `-sS` flag
2. **Port Range Selection**: Efficient use of `-p`, `--top-ports`, `-p-`
3. **Timing Control**: Proper use of `-T4` for lab environments
4. **Output Management**: Always use `-oA` for documentation
5. **UDP Scanning**: Basic understanding of `-sU` limitations
6. **Host Discovery**: Know when to use `-Pn`

### Critical Port Scanning Workflow for eJPT:
```bash
# 1. Network discovery
nmap -sn <lab-network>

# 2. Quick TCP assessment
nmap -sS -T4 -F <target-ip>

# 3. Full TCP port scan
nmap -sS -T4 -p- --open -oA tcp_full <target-ip>

# 4. Selective UDP scan
nmap -sU -T4 --top-ports 100 --open <target-ip>

# 5. Verify interesting ports
nmap -sT -p <interesting-ports> <target-ip>
```

---

## 🚨 Common Port Scanning Issues

### Performance Problems
```bash
# Scan taking too long?
nmap -T4 --min-rate 1000 --max-retries 2 <target-ip>

# Getting timeout errors?
nmap --host-timeout 10m <target-ip>

# Need faster host discovery?
nmap -n -sn <network>  # Skip DNS resolution
```

### Accuracy Issues
```bash
# Ports showing as filtered?
nmap -Pn -sT <target-ip>  # Skip ping, use TCP Connect

# Missing open ports?
nmap -sS -sT -p <port> <target-ip>  # Compare scan methods

# UDP scan showing nothing?
sudo nmap -sU -sV --version-intensity 0 -p <udp-ports> <target-ip>
```

### Access Problems
```bash
# Permission denied for SYN scan?
sudo nmap -sS <target-ip>

# Can't reach target?
nmap -Pn -sT <target-ip>

# Firewall blocking scans?
nmap -f --source-port 53 <target-ip>
```

---

## 📚 Port Scanning Quick Reference

### Most Critical Commands
```bash
# Basic TCP port scan
nmap -sS <target-ip>

# Fast comprehensive scan
nmap -sS -T4 -p- --open <target-ip>

# UDP scan (top ports)
nmap -sU --top-ports 100 <target-ip>

# Document results
nmap -sS -T4 -p- -oA scan_results <target-ip>

# Network discovery
nmap -sn <network-range>

# Skip host discovery
nmap -Pn -sS <target-ip>
```

### Essential Port Scanning Flags
- `-sS` = SYN scan (stealth, requires root)
- `-sT` = TCP Connect scan (no root needed)
- `-sU` = UDP scan
- `-p-` = Scan all 65535 ports
- `-p 1-1000` = Scan port range  
- `--top-ports N` = Scan top N most common ports
- `-T4` = Aggressive timing (fast)
- `-T1` = Slow timing (stealth)
- `--open` = Show only open ports
- `-Pn` = Skip ping (assume host is up)
- `-oA` = Output in all formats

This focused guide provides comprehensive coverage of Nmap port scanning techniques essential for penetration testing and security assessments.
