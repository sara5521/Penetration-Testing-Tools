# 🗄️ Database Service Enumeration with Nmap - Complete Guide

This comprehensive guide includes useful Nmap scripts to enumerate database services across various database systems including MySQL, MSSQL, Oracle, PostgreSQL, MongoDB, and Redis for penetration testing and security assessments.

## 📋 Table of Contents
- [Purpose](#-purpose)
- [Script Summary](#-database-enumeration-scripts-summary)
- [Prerequisites](#-prerequisites)
- [MySQL Enumeration](#-1-mysql-enumeration)
- [Microsoft SQL Server](#-2-microsoft-sql-server-enumeration)
- [Oracle Database](#-3-oracle-database-enumeration)
- [PostgreSQL](#-4-postgresql-enumeration)
- [MongoDB](#-5-mongodb-enumeration)
- [Redis Database](#-6-redis-database-enumeration)
- [Database Discovery](#-7-database-port-scanning-and-discovery)
- [Authentication Testing](#-8-database-authentication-testing)
- [Advanced Techniques](#-9-advanced-database-enumeration)
- [Complete Assessment Workflow](#-complete-database-assessment-workflow)
- [All-in-One Scan](#-all-in-one-scan)
- [Troubleshooting](#-troubleshooting)
- [Security Considerations](#-security-considerations)

---

## 🎯 Purpose

These scripts help you gather important information from database services, such as:
- Database server software and version identification
- Database names and table structures
- User accounts and permissions
- Authentication mechanisms and security settings
- Configuration information and variables
- Default or weak credentials
- Stored procedures and functions
- Data extraction and hash dumping capabilities
- Security vulnerabilities and misconfigurations

---

## 🗂️ Database Enumeration Scripts Summary

| # | Script                    | What it Does                             | Auth Required | Database |
|---|---------------------------|------------------------------------------|---------------|----------|
| 1 | mysql-info                | Extract MySQL server information         | ❌            | MySQL |
| 2 | mysql-databases           | List MySQL databases                     | ✅            | MySQL |
| 3 | mysql-users               | Enumerate MySQL user accounts            | ✅            | MySQL |
| 4 | mysql-empty-password      | Test for empty passwords                 | ❌            | MySQL |
| 5 | mysql-brute               | Brute force MySQL authentication        | ❌            | MySQL |
| 6 | ms-sql-info               | Extract MSSQL server information         | ❌            | MSSQL |
| 7 | ms-sql-tables             | List MSSQL tables                        | ✅            | MSSQL |
| 8 | ms-sql-brute              | Brute force MSSQL authentication        | ❌            | MSSQL |
| 9 | ms-sql-xp-cmdshell        | Execute OS commands via MSSQL            | ✅            | MSSQL |
| 10 | oracle-sid-brute         | Brute force Oracle SID discovery        | ❌            | Oracle |
| 11 | oracle-brute             | Brute force Oracle authentication       | ❌            | Oracle |
| 12 | pgsql-brute              | Brute force PostgreSQL authentication   | ❌            | PostgreSQL |
| 13 | mongodb-info             | Extract MongoDB information              | ❌            | MongoDB |
| 14 | mongodb-databases        | List MongoDB databases                   | ❌            | MongoDB |
| 15 | redis-info               | Extract Redis information                | ❌            | Redis |

---

## 📋 Prerequisites

Before starting, ensure:
- **Target has database services enabled** (various ports)
- **Proper authorization** to test the target
- **Network connectivity** to the target

### Quick Database Port Check:
```bash
# Check all common database ports
nmap -sV -p 1433,3306,5432,1521,27017,6379 <target-ip>

# Check extended database ports
nmap -p 1433,3306,5432,1521,1522,27017,6379,33060,50000 <target-ip>
```

### Standard Database Ports:
- **3306**: MySQL
- **1433**: Microsoft SQL Server
- **5432**: PostgreSQL
- **1521**: Oracle Database
- **27017**: MongoDB
- **6379**: Redis
- **33060**: MySQL X Protocol
- **1522**: Oracle (alternative)

---

## 🔧 Database Service Enumeration Commands

## 🐬 1. MySQL Enumeration
```bash
# Basic MySQL service information
nmap --script mysql-info -p 3306 <target-ip>

# MySQL database enumeration
nmap --script mysql-databases -p 3306 <target-ip>

# MySQL user enumeration
nmap --script mysql-users -p 3306 <target-ip>

# All MySQL scripts
nmap --script mysql-* -p 3306 <target-ip>
```
**Purpose:**

MySQL enumeration helps identify:
- Server version and capabilities
- Available databases
- User accounts and privileges
- Configuration variables
- Security vulnerabilities

**INE Lab Example:**
```bash
nmap --script mysql-info,mysql-empty-password -p 3306 demo.ine.local
```
**Sample Output:**
```bash
PORT     STATE SERVICE
3306/tcp open  mysql

| mysql-info: 
|   Protocol: 10
|   Version: 5.7.25-0ubuntu0.18.04.2
|   Thread ID: 8
|   Capabilities flags: 65535
|   Some Capabilities: SupportsTransactions, LongColumnFlag, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: g\x0F\x7F\x1E\x05l\x0C\x02\x1A>jdVN\x04\x1F\x12\x0C\x7F

| mysql-empty-password: 
|_  root account has empty password
```
**Interpretation:**
- MySQL version 5.7.25 running on Ubuntu
- Root account has empty password (critical security issue)
- Server supports transactions and multiple capabilities
- Protocol version 10 in use

### MySQL Authentication and Access Testing:
```bash
# MySQL anonymous access check
nmap --script mysql-empty-password -p 3306 <target-ip>

# MySQL brute force
nmap --script mysql-brute -p 3306 <target-ip>

# MySQL brute force with custom wordlists
nmap --script mysql-brute --script-args userdb=users.txt,passdb=passwords.txt -p 3306 <target-ip>

# MySQL authentication with credentials
nmap --script mysql-databases --script-args mysqluser=root,mysqlpass=password -p 3306 <target-ip>
```

### MySQL Security Assessment:
```bash
# MySQL configuration audit
nmap --script mysql-audit -p 3306 <target-ip>

# MySQL variable enumeration
nmap --script mysql-variables -p 3306 <target-ip>

# MySQL vulnerability assessment
nmap --script mysql-vuln-* -p 3306 <target-ip>

# MySQL hash dumping
nmap --script mysql-dump-hashes --script-args mysqluser=root,mysqlpass= -p 3306 <target-ip>
```

---

## 🗄️ 2. Microsoft SQL Server Enumeration
```bash
# MSSQL service information
nmap --script ms-sql-info -p 1433 <target-ip>

# MSSQL instance discovery
nmap --script ms-sql-discover -p 1433 <target-ip>

# MSSQL configuration
nmap --script ms-sql-config -p 1433 <target-ip>

# All MSSQL scripts
nmap --script ms-sql-* -p 1433 <target-ip>
```
**Purpose:**

MSSQL enumeration reveals:
- Server version and instance information
- Database names and structures
- User accounts and roles
- Configuration settings
- Command execution capabilities

**INE Lab Example:**
```bash
nmap --script ms-sql-info,ms-sql-empty-password -p 1433 demo.ine.local
```
**Sample Output:**
```bash
PORT     STATE SERVICE
1433/tcp open  ms-sql-s

| ms-sql-info: 
|   Windows server name: WIN-SERVER2016
|   Version: 
|     name: Microsoft SQL Server 2016 RTM
|     number: 13.00.1601.05
|     Product: Microsoft SQL Server 2016
|     Service pack level: RTM
|     Post-SP patches applied: false
|   TCP port: 1433
|   Named pipe: \\WIN-SERVER2016\pipe\sql\query
|_  Clustered: false

| ms-sql-empty-password: 
|   sa:<empty> => Login Success
|_  ERROR: No users found with empty passwords
```
**Interpretation:**
- SQL Server 2016 RTM on Windows
- Default 'sa' account has empty password
- Named pipe available for local connections
- Non-clustered instance

### MSSQL Authentication and Access:
```bash
# MSSQL empty password check
nmap --script ms-sql-empty-password -p 1433 <target-ip>

# MSSQL brute force
nmap --script ms-sql-brute -p 1433 <target-ip>

# MSSQL brute force with domain accounts
nmap --script ms-sql-brute --script-args mssql.domain=DOMAIN -p 1433 <target-ip>

# MSSQL with specific credentials
nmap --script ms-sql-tables --script-args mssql.username=sa,mssql.password=password -p 1433 <target-ip>
```

### MSSQL Advanced Enumeration:
```bash
# MSSQL database enumeration
nmap --script ms-sql-dac -p 1433 <target-ip>

# MSSQL table enumeration
nmap --script ms-sql-tables -p 1433 <target-ip>

# MSSQL command execution
nmap --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=pass,ms-sql-xp-cmdshell.cmd="whoami" -p 1433 <target-ip>

# MSSQL hash extraction
nmap --script ms-sql-hashdump --script-args mssql.username=sa,mssql.password=pass -p 1433 <target-ip>
```

---

## 🔶 3. Oracle Database Enumeration
```bash
# Oracle TNS listener enumeration
nmap --script oracle-tns-version -p 1521,1522,1525 <target-ip>

# Oracle SID enumeration
nmap --script oracle-sid-brute -p 1521 <target-ip>

# Oracle brute force authentication
nmap --script oracle-brute -p 1521 <target-ip>

# Oracle comprehensive enumeration
nmap --script oracle-* -p 1521 <target-ip>
```
**Purpose:**

Oracle enumeration discovers:
- TNS listener version and status
- Oracle SID (System Identifier) values
- Database instances and services
- Authentication mechanisms
- User accounts and privileges

**Sample Oracle Output:**
```bash
| oracle-tns-version: 
|   Version: 11.2.0.2.0
|   Service Banner: TNSLSNR for Linux: Version 11.2.0.2.0
|_  Service Banner: Oracle Database 11g Release 11.2.0.2.0

| oracle-sid-brute: 
|   XE
|   ORCL
|_  PROD
```

### Oracle Security Assessment:
```bash
# Oracle TNS listener security
nmap --script oracle-tns-version,oracle-sid-brute -p 1521 <target-ip>

# Oracle brute force with specific SID
nmap --script oracle-brute --script-args oracle-brute.sid=XE -p 1521 <target-ip>

# Oracle enumeration with credentials
nmap --script oracle-enum-users --script-args oracleuser=system,oraclepass=manager -p 1521 <target-ip>
```

---

## 🐘 4. PostgreSQL Enumeration
```bash
# PostgreSQL service information
nmap -sV -p 5432 <target-ip>

# PostgreSQL brute force authentication
nmap --script pgsql-brute -p 5432 <target-ip>

# PostgreSQL brute force with custom lists
nmap --script pgsql-brute --script-args userdb=users.txt,passdb=passwords.txt -p 5432 <target-ip>

# PostgreSQL default credentials testing
nmap --script pgsql-brute --script-args brute.firstonly=true -p 5432 <target-ip>
```
**Purpose:**

PostgreSQL assessment identifies:
- Server version and configuration
- Authentication methods
- Database accessibility
- User account enumeration

---

## 🍃 5. MongoDB Enumeration
```bash
# MongoDB service detection
nmap -sV -p 27017 <target-ip>

# MongoDB information gathering
nmap --script mongodb-info -p 27017 <target-ip>

# MongoDB database enumeration
nmap --script mongodb-databases -p 27017 <target-ip>

# MongoDB brute force
nmap --script mongodb-brute -p 27017 <target-ip>
```
**Purpose:**

MongoDB enumeration reveals:
- MongoDB version and build information
- Available databases and collections
- Authentication status
- Administrative access

**Sample MongoDB Output:**
```bash
| mongodb-info: 
|   MongoDB Build info
|     Version: 3.6.3
|     GitVersion: 9586e557d54ef70f9ca4b43c26892cd55257e1a5
|     OpenSSLVersion: OpenSSL 1.0.1t  3 May 2016
|     JavaScriptEngine: mozjs
|     Architecture: x86_64
|   Server status  
|     Current time: 2024-01-15T10:30:00.000Z
|     Hostname: mongo-server
|_    Uptime: 3600 seconds

| mongodb-databases: 
|   totalSize: 83886080
|   databases
|     0
|       name: admin
|       sizeOnDisk: 32768
|       empty: false
|     1
|       name: config
|       sizeOnDisk: 32768
|       empty: false
|     2
|       name: local
|       sizeOnDisk: 32768
|_      empty: false
```

---

## 🔴 6. Redis Database Enumeration
```bash
# Redis information gathering
nmap --script redis-info -p 6379 <target-ip>

# Redis brute force
nmap --script redis-brute -p 6379 <target-ip>

# Redis command execution (if no auth)
nmap --script redis-info -p 6379 <target-ip>
```
**Purpose:**

Redis assessment discovers:
- Redis version and configuration
- Authentication requirements
- Database information
- Security settings

---

## 📊 7. Database Port Scanning and Discovery
```bash
# All common database ports
nmap -sV -p 1433,3306,5432,1521,27017,6379 <target-ip>

# Extended database port scan
nmap -p 1433,3306,5432,1521,1522,1525,27017,27018,6379,33060,50000 <target-ip>

# Alternative database ports
nmap -p 2433,3307,5433,1526,27019,6380 <target-ip>

# High-port database services
nmap -p 33060,50000,50001,8086,9042 <target-ip>
```

### Database Service Discovery Workflow:
```bash
# Database administration ports
nmap -p 8080,8443,9000,10000 <target-ip> # Web-based DB admin

# Embedded database ports
nmap -p 1527,9001,9002 <target-ip>

# NoSQL databases
nmap -p 27017,27018,27019,6379,8086,9042 <target-ip>
```

---

## 🔑 8. Database Authentication Testing
```bash
# Empty password testing across databases
nmap --script mysql-empty-password,ms-sql-empty-password -p 3306,1433 <target-ip>

# Brute force authentication across multiple databases
nmap --script mysql-brute,ms-sql-brute,oracle-brute,mongodb-brute --script-args brute.firstonly=true -p 1433,3306,1521,27017 <target-ip>

# Default credentials testing
nmap --script mysql-brute,ms-sql-brute --script-args userdb=default_users.txt,passdb=default_passwords.txt -p 3306,1433 <target-ip>

# Anonymous access testing
nmap --script mysql-info,mongodb-info,redis-info -p 3306,27017,6379 <target-ip>
```

---

## 🛠️ 9. Advanced Database Enumeration

### Database Content Extraction:
```bash
# MySQL hash extraction
nmap --script mysql-dump-hashes --script-args mysqluser=root,mysqlpass= -p 3306 <target-ip>

# MSSQL hash dumping
nmap --script ms-sql-hashdump --script-args mssql.username=sa,mssql.password=pass -p 1433 <target-ip>

# Database and table enumeration
nmap --script mysql-databases,mysql-users,ms-sql-tables --script-args mysqluser=root,mysqlpass=,mssql.username=sa,mssql.password= -p 3306,1433 <target-ip>
```

### Command Execution via Databases:
```bash
# MSSQL command execution via xp_cmdshell
nmap --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=pass,ms-sql-xp-cmdshell.cmd="net user" -p 1433 <target-ip>

# MySQL UDF command execution (if available)
nmap --script mysql-audit --script-args mysqluser=root,mysqlpass= -p 3306 <target-ip>
```

### Configuration Assessment:
```bash
# MySQL configuration variables
nmap --script mysql-variables --script-args mysqluser=root,mysqlpass= -p 3306 <target-ip>

# MSSQL configuration settings
nmap --script ms-sql-config --script-args mssql.username=sa,mssql.password= -p 1433 <target-ip>

# Oracle instance configuration
nmap --script oracle-tns-version -p 1521 <target-ip>
```

---

## 🎯 Complete Database Assessment Workflow

### Phase 1: Service Discovery
```bash
# Step 1: Database port discovery
nmap -sS -p 1433,3306,5432,1521,27017,6379 <target>

# Step 2: Service version detection
nmap -sV -p 1433,3306,5432,1521,27017,6379 <target>

# Step 3: Basic service information
nmap --script mysql-info,ms-sql-info,oracle-tns-version,mongodb-info,redis-info -p 1433,3306,1521,27017,6379 <target>
```

### Phase 2: Authentication Testing
```bash
# Step 4: Empty password testing
nmap --script mysql-empty-password,ms-sql-empty-password -p 3306,1433 <target>

# Step 5: Default credentials testing
nmap --script mysql-brute,ms-sql-brute,oracle-brute --script-args brute.firstonly=true -p 1433,3306,1521 <target>

# Step 6: Anonymous access testing
nmap --script mongodb-databases,redis-info -p 27017,6379 <target>
```

### Phase 3: Database Enumeration
```bash
# Step 7: Database and table enumeration (with credentials)
nmap --script mysql-databases,ms-sql-tables --script-args mysqluser=root,mysqlpass=,mssql.username=sa,mssql.password= -p 3306,1433 <target>

# Step 8: User enumeration
nmap --script mysql-users --script-args mysqluser=root,mysqlpass= -p 3306 <target>

# Step 9: Configuration assessment
nmap --script mysql-variables,ms-sql-config --script-args mysqluser=root,mysqlpass=,mssql.username=sa,mssql.password= -p 3306,1433 <target>
```

---

## 🔁 All-in-One Scan

### Basic Database Discovery (no authentication)
```bash
nmap -sV -p 1433,3306,5432,1521,27017,6379 --script "mysql-info,ms-sql-info,oracle-tns-version,mongodb-info,redis-info" <target-ip>
```

### Authentication Testing Scan
```bash
nmap -p 1433,3306,1521,27017 --script "mysql-empty-password,ms-sql-empty-password,mysql-brute,ms-sql-brute,oracle-brute" --script-args brute.firstonly=true <target-ip>
```

### Comprehensive Database Assessment
```bash
nmap --script mysql-*,ms-sql-*,oracle-*,mongodb-*,redis-*,pgsql-* -p 1433,3306,5432,1521,27017,6379 <target-ip>
```

### Database Security Assessment
```bash
nmap -p 1433,3306,1521 --script "mysql-vuln-*,ms-sql-xp-cmdshell,oracle-sid-brute" --script-args mysqluser=root,mysqlpass=,mssql.username=sa,mssql.password= <target-ip>
```

---

## 📊 Real Lab Examples

### Example 1: MySQL Server Comprehensive Assessment
```bash
# Target: MySQL server with potential security issues

# Service detection
nmap -sV -p 3306 192.168.1.100

# Basic information gathering
nmap --script mysql-info,mysql-empty-password -p 3306 192.168.1.100

# Authentication testing
nmap --script mysql-brute --script-args brute.firstonly=true -p 3306 192.168.1.100

# Database enumeration (with found credentials)
nmap --script mysql-databases,mysql-users --script-args mysqluser=root,mysqlpass= -p 3306 192.168.1.100

# Security assessment
nmap --script mysql-variables,mysql-audit --script-args mysqluser=root,mysqlpass= -p 3306 192.168.1.100
```

**Expected Output:**
```
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.7.25-0ubuntu0.18.04.2

| mysql-empty-password: 
|_  root account has empty password

| mysql-databases: 
|   information_schema
|   mysql
|   performance_schema
|   sys
|   wordpress
|_  ecommerce

| mysql-users: 
|   root
|   mysql.session
|   mysql.sys
|   debian-sys-maint
|_  webapp_user
```

### Example 2: MSSQL Server with Command Execution
```bash
# Target: MSSQL server with xp_cmdshell enabled

# Service discovery and information
nmap -sV --script ms-sql-info,ms-sql-empty-password -p 1433 192.168.1.200

# Table enumeration
nmap --script ms-sql-tables --script-args mssql.username=sa,mssql.password= -p 1433 192.168.1.200

# Command execution testing
nmap --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=,ms-sql-xp-cmdshell.cmd="whoami" -p 1433 192.168.1.200

# Hash extraction
nmap --script ms-sql-hashdump --script-args mssql.username=sa,mssql.password= -p 1433 192.168.1.200
```

### Example 3: Multi-Database Environment
```bash
# Target: Server with multiple database services

# Multi-database discovery
nmap -sV -p 1433,3306,5432,1521,27017 192.168.1.50

# Authentication testing across all services
nmap --script mysql-empty-password,ms-sql-empty-password,mongodb-databases,redis-info -p 3306,1433,27017,6379 192.168.1.50

# Comprehensive enumeration
nmap --script mysql-databases,ms-sql-tables,mongodb-databases --script-args mysqluser=root,mysqlpass=,mssql.username=sa,mssql.password= -p 3306,1433,27017 192.168.1.50
```

---

## 🛠️ Troubleshooting

### Common Issues and Solutions:

| Issue | Symptoms | Solution |
|-------|----------|----------|
| **Connection Refused** | `Connection refused` | Check if database service is running |
| **Authentication Failed** | Login failures | Verify credentials and account status |
| **Access Denied** | Permission errors | Check user privileges and database policies |
| **Timeout Issues** | Scripts hang or timeout | Use `--script-args timeout=10` |
| **Empty Results** | No databases found | Verify authentication and permissions |

### Common Error Messages:
- `mysql: Access denied for user 'root'@'host'` → Invalid credentials or host restrictions
- `mssql: Login failed for user 'sa'` → Invalid credentials or disabled account
- `oracle: ORA-12541: TNS:no listener` → Oracle listener not running
- `mongodb: Authentication failed` → MongoDB requires authentication
- `redis: NOAUTH Authentication required` → Redis password required

### Performance Optimization:
```bash
# Faster database scanning
nmap -T4 --script mysql-info,ms-sql-info -p 3306,1433 <target>

# Optimized brute force
nmap --script mysql-brute,ms-sql-brute --script-args brute.threads=2,brute.delay=1 -p 3306,1433 <target>

# Custom timeout settings
nmap --script mysql-*,ms-sql-* --script-args timeout=10 -p 3306,1433 <target>
```

---

## ⚠️ Security Considerations

### Legal and Ethical Guidelines:
- **Always get proper written authorization** before testing
- **Only test systems you own** or have explicit permission to test
- **Database testing can impact performance** - be cautious with timing
- **Follow responsible disclosure** if vulnerabilities are found

### Operational Security (OpSec):
- **Database connections generate extensive logs** - be aware of detection
- **Brute force attacks can lock accounts** - use with extreme caution
- **Command execution leaves traces** - consider forensic implications
- **Hash extraction is highly invasive** - use only with proper authorization

### Detection Indicators:
Target systems may detect database enumeration through:
- **Database Logs**: Connection attempts and authentication failures
- **Security Monitoring**: Unusual database access patterns
- **Failed Authentication**: Multiple login attempts triggering alerts
- **Command Execution**: OS-level logging of database-initiated commands

### Risk Assessment:
| Finding | Risk Level | Recommendation |
|---------|------------|----------------|
| Empty Database Passwords | 🔴 Critical | Set strong passwords immediately |
| Default Credentials | 🔴 Critical | Change all default accounts |
| Command Execution Enabled | 🔴 Critical | Disable xp_cmdshell and similar features |
| Unauthorized Database Access | 🟠 High | Implement proper access controls |
| Information Disclosure | 🟡 Medium | Restrict database enumeration |

---

## 🎯 eJPT Focus Points

### Essential Database Commands for eJPT:
```bash
# Multi-database discovery
nmap -sV -p 1433,3306,5432,1521,27017,6379 <target>

# Empty password testing
nmap --script mysql-empty-password,ms-sql-empty-password -p 3306,1433 <target>

# Basic database information
nmap --script mysql-info,ms-sql-info -p 3306,1433 <target>

# Database brute force
nmap --script mysql-brute,ms-sql-brute --script-args brute.firstonly=true -p 3306,1433 <target>
```

### Key Database Assessment Areas for eJPT:
1. **Service Discovery**: Identify database services and versions
2. **Authentication Testing**: Test for weak or default credentials
3. **Information Gathering**: Extract database names, tables, and users
4. **Privilege Escalation**: Use database access for system compromise
5. **Data Extraction**: Retrieve sensitive information and password hashes
6. **Command Execution**: Leverage database features for OS command execution

### Common eJPT Database Scenarios:
- **MySQL Root Access**: Empty root password leading to database compromise
- **MSSQL xp_cmdshell**: Command execution via SQL Server
- **MongoDB Exposure**: Unprotected MongoDB instances with sensitive data
- **Oracle TNS Enumeration**: SID brute forcing and authentication bypass
- **Database Credential Discovery**: Using database access to find application credentials

### Critical Database Information for Further Testing:
- **Valid Credentials**: Database usernames and passwords for system access
- **Database Content**: Sensitive data, user information, application secrets
- **Configuration Details**: Database settings revealing security weaknesses
- **Hash Values**: Password hashes for offline cracking attacks
- **Command Execution**: Database-to-OS privilege escalation paths

---

## 🔗 Integration with Other Tools

After Nmap database enumeration, follow up with:
- **mysql/mysqldump**: MySQL client and data extraction tools
- **sqlcmd**: Microsoft SQL Server command-line client
- **sqlmap**: Automated SQL injection testing and exploitation
- **MongoDB Compass**: MongoDB GUI client for data exploration
- **DBeaver**: Universal database client for multiple database types
- **Metasploit**: Database exploitation modules and post-exploitation
- **NoSQLMap**: NoSQL database assessment and exploitation tool
- **hydra**: Advanced database authentication brute forcing

### Database Security Testing Workflow:
```bash
# 1. Nmap database discovery and enumeration
nmap --script mysql-*,ms-sql-*,oracle-*,mongodb-* -p 1433,3306,1521,27017 <target>

# 2. Extract discovered credentials for further testing
grep -E "(Valid credentials|empty password|login successful)" database_results.txt

# 3. Direct database access with discovered credentials
mysql -h <target> -u root -p
sqlcmd -S <target> -U sa -P <password>

# 4. Data extraction and analysis
mysqldump -h <target> -u root -p --all-databases > database_dump.sql

# 5. Advanced exploitation with Metasploit
use auxiliary/scanner/mysql/mysql_login
use exploit/windows/mssql/mssql_payload
```

---

## 📝 Output and Documentation

### Saving Database Assessment Results:
```bash
# Comprehensive database assessment
nmap --script mysql-*,ms-sql-*,oracle-*,mongodb-* -oA database_enumeration -p 1433,3306,5432,1521,27017 <target>

# MySQL-specific assessment
nmap --script mysql-* -oN mysql_assessment.txt -p 3306 <target>

# MSSQL vulnerability scan
nmap --script ms-sql-* -oX mssql_scan.xml -p 1433 <target>
```

### Results Processing and Analysis:
```bash
# Extract database names and structures
grep -A 20 "mysql-databases\|ms-sql-tables\|mongodb-databases" database_enumeration.nmap

# Find authentication issues
grep -i "empty password\|no password\|login successful" database_enumeration.nmap
