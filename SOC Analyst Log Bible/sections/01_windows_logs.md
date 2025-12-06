# **Section 1 — Windows Security Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

Windows Security logs are the backbone of Windows-based intrusion detection.  
Almost every attacker action generates at least one of these event IDs — often many.  
This section explains **what each event means, why it matters, how attackers abuse it,
MITRE ATT&CK mapping, real log examples, and detection logic for SIEM.**

---

# 🧭 **TABLE OF CONTENTS**
- [4624 — Successful Logon](#4624--successful-logon)
- [4625 — Failed Logon](#4625--failed-logon)
- [4672 — Special Privileges Assigned](#4672--special-privileges-assigned)
- [4688 — New Process Created](#4688--new-process-created)
- [4648 — Logon with Explicit Credentials](#4648--logon-with-explicit-credentials)
- [4634 — Logoff](#4634--logoff)
- [4768 — Kerberos Authentication (TGT Request)](#4768--kerberos-authentication-tgt-request)
- [4769 — Kerberos Service Ticket (TGS)](#4769--kerberos-service-ticket-tgs)
- [4776 — NTLM Authentication](#4776--ntlm-authentication)
- [7045 — New Service Installed (System Log)](#7045--a-service-was-installed-system-log)

---

# ---
# ## **4624 — Successful Logon**
### **Meaning**
A user successfully authenticated.

### **Why it matters**
Critical for:
- detecting lateral movement  
- identifying compromised accounts  
- spotting RDP / SMB / remote access behavior  
- mapping attacker movement between hosts  

### **Key Field — LogonType**
| LogonType | Meaning | SOC Relevance |
|----------|---------|----------------|
| **2** | Interactive | Physical login (rare in servers) |
| **3** | Network | Lateral movement / SMB |
| **4** | Batch | Scheduled Tasks |
| **5** | Service | Service account logon |
| **7** | Unlock | Not usually relevant |
| **8** | NetworkCleartext | Credentials sent in cleartext |
| **10** | RDP | Remote Desktop activity |
| **11** | CachedInteractive | Offline logon |

### **Red Flags**
- Unexpected RDP logon type **10**  
- Service accounts logging on interactively  
- Logon from new / foreign IP  
- Privileged accounts logging on outside working hours  

### **MITRE ATT&CK**
- **T1078 – Valid Accounts**  
- **T1021 – Remote Services**

### **Real Attacker Log Example**
```
EventID: 4624
LogonType: 3
AccountName: administrator
AccountDomain: CORP
WorkstationName: FINANCE-01
SourceNetworkAddress: 10.11.5.22
LogonProcess: NtLmSsp
AuthenticationPackage: NTLM
```

### **What to Look For**
- Account used across multiple hosts within minutes  
- Source IP talking to many machines → lateral movement spidering  
- NTLM logons where Kerberos is expected  
- LogonProcess = “Seclogon” → UAC bypass / service abuse  

### **SIEM Detection Query (KQL)**
```kql
SecurityEvent
| where EventID == 4624
| where LogonType in (3, 10)
| summarize count() by AccountName, IpAddress, Computer
| order by count_ desc
```

---

# ---
# ## **4625 — Failed Logon**
### **Meaning**
Authentication attempt failed.

### **Why it matters**
Primary signal for:
- **brute force attacks**
- **password spraying**
- credential stuffing  
- attacker enumeration attempts  

### **Key Fields**
- **FailureReason** → unknown user, bad password, locked account  
- **SubStatus** → 0xC000006A, 0xC0000064, etc.  
- **LogonType** → especially network (3) or RDP (10)

### **MITRE ATT&CK**
- **T1110 – Brute Force**

### **Real Attacker Log Example**
```
EventID: 4625
LogonType: 10
AccountName: admin
SourceNetworkAddress: 185.244.25.11
FailureReason: Unknown user name or bad password
SubStatus: 0xC000006A
```

### **What to Look For**
- Many failures from a single external IP  
- Many accounts being tried → password spray  
- Attempts on dormant or high-privilege accounts  
- Failures immediately followed by a **4624 success**  

### **SIEM Detection (Password Spraying)**
```kql
SecurityEvent
| where EventID == 4625
| summarize Attempts = count() by AccountName, IPAddress
| where Attempts > 10
```

---

# ---
# ## **4672 — Special Privileges Assigned**
### **Meaning**
A user logged on and received **admin privileges** (SeDebug, SeImpersonate, SeTcbPrivilege, etc.)

### **Why it matters**
This is one of the most important escalation indicators.

Used for detecting:
- privilege escalation  
- compromised admin accounts  
- token manipulation abuse  
- service account privilege anomalies  

### **MITRE ATT&CK**
- **T1068 – Privilege Escalation**
- **T1134 – Access Token Manipulation**

### **Real Attacker Example**
```
EventID: 4672
AccountName: workstation$
Privileges: SeDebugPrivilege, SeImpersonatePrivilege
```

### **What to Look For**
- Machine accounts receiving privileges (shouldn’t happen)  
- Normal users suddenly gaining admin privileges  
- Privileges assigned right after a suspicious 4624 logon  

### **SIEM Alert Logic**
```kql
SecurityEvent
| where EventID == 4672
| where AccountName !contains "$"  // exclude machine accounts
| extend Privs = tostring(Privileges)
| where Privs contains "SeDebugPrivilege" or Privs contains "SeImpersonatePrivilege"
```

---

# ---
# ## **4688 — New Process Created**
### **Meaning**
A process was created with command-line arguments.

### **Why it matters**
The **#1 most valuable Windows event** for malware and attacker detection.

Detects:
- malware execution  
- living-off-the-land (LOLBins)  
- suspicious PowerShell/cmd usage  
- phishing → payload execution  
- lateral movement tools  

### **Important Fields**
- **NewProcessName**
- **CommandLine**
- **ParentProcessName**
- **TokenElevationType**

### **MITRE ATT&CK**
- **T1059 – Command Execution**
- **T1204 – User Execution**
- **T1105 – C2 Communications**

### **Common Malicious Patterns**
| Parent | Child | Meaning |
|--------|--------|----------|
| winword.exe | powershell.exe | Phishing payload executed |
| outlook.exe | cmd.exe | Malicious attachment |
| browser.exe | certutil.exe | Downloading malware |
| explorer.exe | rundll32.exe | DLL execution / LOLBin |
| services.exe | cmd.exe | Remote execution |

### **Real Attacker Example**
```
EventID: 4688
NewProcessName: C:\Windows\System32\cmd.exe
CommandLine: cmd.exe /c powershell -enc JABXAG...
ParentProcess: winword.exe
```

### **What to Look For**
- Base64 encoded commands (`-enc`)  
- Suspicious LOLBins: `rundll32, regsvr32, mshta, powershell`  
- Child processes spawned by Office applications  
- Downloads:
  - `Invoke-WebRequest`
  - `certutil -urlcache -split -f`
  - `bitsadmin`  

### **SIEM Detection**
```kql
SecurityEvent
| where EventID == 4688
| where CommandLine contains "-enc" 
    or CommandLine contains "DownloadString"
    or ParentProcessName in ("winword.exe","excel.exe","outlook.exe")
```

---

# ---
# ## **4648 — Logon with Explicit Credentials**
### **Meaning**
A process attempted to authenticate using **explicit credentials** (e.g. runas, remote execution tools).

### **Why it matters**
Attackers often use:
- Pass-the-Credential  
- runas abuse  
- lateral movement tools (PsExec, WMI, WinRM)

### **MITRE ATT&CK**
- **T1550 – Use of Stolen Credentials**

### **Real Log Example**
```
EventID: 4648
AccountName: user1
ProcessName: C:\Windows\System32\psexec.exe
TargetServer: SERVER01
```

### **Detection Notes**
Flag:
- psexec.exe, wmiprvse.exe, winrm → credential use  
- explicit credentials used outside normal hours  

---

# ---
# ## **4634 — Logoff**
### **Meaning**
A session ended.

### **Why it matters**
Not suspicious alone — but important for:
- session baselining  
- correlating logon durations  
- detecting short bursts of authentication (bruteforce → success → logoff)  

---

# ---
# ## **4768 — Kerberos Authentication (TGT Request)**
### **Meaning**
User requested a **Kerberos Ticket Granting Ticket (TGT).**

### **Why it matters**
Detect:
- Pass-the-Ticket  
- Golden Ticket usage  
- Kerberoasting patterns  

### **MITRE ATT&CK**
- **T1550.003 – Kerberos Tickets**

### **Red Flags**
- High volume TGT requests  
- Requests using disabled accounts  
- Requests from non-domain machines  

---

# ---
# ## **4769 — Kerberos Service Ticket (TGS) Request**
### **Meaning**
User requested access to a specific Kerberos service.

### **Why it matters**
Kerberoasting relies on:
- requesting service tickets for accounts with SPNs  

### **MITRE ATT&CK**
- **T1558.003 – Kerberoasting**

### **Red Flags**
- Service tickets requested for many SPNs quickly  
- Unusual user requesting privileged SPNs  

---

# ---
# ## **4776 — NTLM Authentication**
### **Meaning**
Windows attempted to validate credentials using NTLM instead of Kerberos.

### **Why it matters**
NTLM is less secure and used for:
- lateral movement  
- pass-the-hash  
- fallback authentication  

### **Red Flags**
- NTLM used where Kerberos is expected  
- NTLM failures followed by success  

---

# ---
# ## **7045 — A Service Was Installed (System Log)**
### **Meaning**
A new Windows service was installed.

### **Why it matters**
Attackers use services for:
- **persistence**
- **remote command execution**
- **malware startup**

### **MITRE ATT&CK**
- **T1543 – Create or Modify System Process**

### **Real Attacker Example**
```
ServiceName: winupdate
BinaryPath: C:\Users\Public\svhost.exe
ServiceType: user mode service
```

### **Red Flags**
- Services installed in:
  - AppData
  - Temp
  - Public folders  
- Suspicious names: updater, sysmon, winhlp, winhelp  
- Non-admin users creating services  

### **SIEM Detection**
```kql
Event
| where EventID == 7045
| where BinaryPath contains "AppData" or BinaryPath contains "Public"
```

---

# 🎯 **End of Section 1**
This section gives complete SOC-ready detection coverage for core Windows Security and System logs.

Next:  
➡️ **Section 2 — Sysmon Logs**  
