# **Section 4 — Windows Defender Logs**
_Comprehensive SOC, DFIR & Detection Engineering Reference_

Microsoft Defender is the most widely deployed endpoint security control in Windows environments.  
Its logs provide essential visibility into malware detection, remediation, tampering, and behavioral blocking.

This section covers the most important Windows Defender event IDs, how attackers trigger them, and how SOC teams detect evasion attempts.

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Windows Defender Logging](#overview-of-windows-defender-logging)
- [Event ID 1116 — Malware Detected](#event-id-1116--malware-detected)
- [Event ID 1117 — Malware Action Taken](#event-id-1117--malware-action-taken)
- [Event ID 5007 — Configuration Changed](#event-id-5007--configuration-changed)
- [Event ID 5010 — Behavior Block](#event-id-5010--behavior-block)
- [Common Evasion Techniques](#common-evasion-techniques)
- [Key Defender Log Locations](#key-defender-log-locations)
- [SIEM Detection Queries](#siem-detection-queries)

---

# ---
# ## **Overview of Windows Defender Logging**

Windows Defender logs appear in:

### **Event Viewer**
- `Applications and Services Logs / Microsoft / Windows / Windows Defender / Operational`

### **Key log types**
- **Malware detection**
- **Threat remediation actions**
- **Tamper protection events**
- **Configuration changes**
- **Behavior-based detections**
- **Cloud-delivered protection logs**

Defender is highly integrated with Windows, meaning **attackers cannot easily avoid generating these logs**—unless they disable Defender first, which itself generates logs.

---

# ---
# ## **Event ID 1116 — Malware Detected**
### **Meaning**
Defender detected a malicious file, behavior, or script, but has not yet taken action.

### **Why It Matters**
This event is the earliest warning of:
- malware execution  
- suspicious downloads  
- phishing payload activation  
- ransomware pre-stages  
- malicious PowerShell scripts  

### **MITRE ATT&CK**
- **T1204 — User Execution**
- **T1059 — Command Execution**
- **T1027 — Obfuscated/Encrypted Malware**

### **Real Attacker Example**
```
EventID: 1116
Threat Name: Trojan:Win32/Injector
Path: C:\Users\Public\update.exe
User: CORP\john.doe
Detection Source: Real-Time Protection
Severity: Severe
```

### **What to Look For**
- Defender detecting files in:
  - Temp
  - AppData
  - Public
  - Downloads
- Detections triggered by Office or browser processes
- Repeated detections → persistent infection

### **Detection Notes**
1116 alone = detection  
1116 followed by no 1117 = Defender failed to act  

---

# ---
# ## **Event ID 1117 — Malware Action Taken**
### **Meaning**
Defender attempted remediation or quarantine.

### **Possible Actions**
- Blocked  
- Quarantined  
- Removed  
- Failed to remove  

### **MITRE ATT&CK**
- **T1562.001 — Disable or Modify Tools**
- **T1490 — Inhibit System Recovery**

### **Real Example**
```
EventID: 1117
Threat: Trojan:Win32/Agent
Action: Quarantined
Status: Succeeded
```

### **Red Flags**
- **Action: Failed** — malware persisted  
- Repeated detections → malware regenerating  
- System32 or Program Files paths → high severity  
- Removal failed due to file lock (often a sign of active malware)

---

# ---
# ## **Event ID 5007 — Configuration Changed**
### **Meaning**
Windows Defender settings were modified.

### **Why It Matters**
Each configuration change may indicate:

### **1) Legit admin configuration**  
→ software deployment, policy changes

### **2) ATTACKER TAMPERING** 🔥  
→ trying to disable protection before launching malware

### **Common malicious attempts**
- Disable Real-Time Protection  
- Disable IOAV (download scanning)  
- Disable Cloud Protection  
- Exclude folders (AppData, Temp)  
- Disable reporting/telemetry  

### **MITRE ATT&CK**
- **T1562 — Defense Evasion**
- **T1112 — Modify Registry**  

### **Real Attacker Example**
```
EventID: 5007
OldValue: DisableRealtimeMonitoring = 0
NewValue: DisableRealtimeMonitoring = 1
```

### **Detection Notes**
Immediate SOC action recommended if:
- Real-time protection disabled  
- Cloud-delivered protection disabled  
- Exclusions added to suspicious folders  

---

# ---
# ## **Event ID 5010 — Behavior Block**
### **Meaning**
Defender blocked suspicious behavior even if no malware file was detected.

### **Why It Matters**
This is Defender's **behavioral engine**:

Catches:
- ransomware encryption  
- credential dumping patterns  
- lateral movement tools  
- exploit payloads  
- macro-based commands  
- PowerShell abuse  

### **MITRE ATT&CK**
Depends on behavior, most often:  
- **T1059 — Command Execution**  
- **T1003 — Credential Dumping**  
- **T1486 — Ransomware Encryption**

### **Real Example**
```
EventID: 5010
Behavior: Suspicious PowerShell Command
Blocked Process: powershell.exe
CommandLine: powershell -nop -w hidden -enc ...
```

### **Red Flags**
- Behavior blocks from Office processes  
- Behavior blocks followed by PowerShell 4104 events  
- Multiple behavior blocks in short time  

---

# ---
# ## **Common Evasion Techniques**
Attackers commonly attempt:

### **1️⃣ Disabling Defender**
```
Set-MpPreference -DisableRealtimeMonitoring $true
```

### **2️⃣ Adding Exclusions**
```
Set-MpPreference -ExclusionPath "C:\Users\Public"
```

### **3️⃣ Stopping the Service**  
(rarely succeeds due to tamper protection)

### **4️⃣ Modifying registry keys**

### **5️⃣ Killing MsMpEng.exe**
(often unsuccessful)

### **6️⃣ Dropping unsigned binaries into excluded folders**

Every one of these attempts triggers logs (mostly 5007).

---

# ---
# ## **Key Defender Log Locations**

### **Operational log**
```
Applications and Services Logs/
  Microsoft/
    Windows/
      Windows Defender/
        Operational
```

### **Filtering tips**
Filter by:

| Event ID | Meaning |
|----------|---------|
| **1116** | detection |
| **1117** | action taken |
| **5007** | config changed |
| **5010** | behavioral block |

---

# ---
# ## **SIEM Detection Queries**

### **KQL — Detect Defender Disabled**
```kql
Defender
| where EventID == 5007
| where NewValue contains "DisableRealtimeMonitoring = 1"
```

### **KQL — Malware Detected but Not Removed**
```kql
Defender
| where EventID == 1116
| join kind=leftanti (
    Defender | where EventID == 1117
) on ThreatID
```

### **KQL — Behavior-Based Block**
```kql
Defender
| where EventID == 5010
| summarize count() by Behavior, Computer, ProcessName
```

### **KQL — Suspicious Exclusions**
```kql
Defender
| where EventID == 5007
| where NewValue contains "ExclusionPath"
| where NewValue contains_any ("AppData","Temp","Public")
```

---

# 🎯 **End of Section 4**
This section enables SOC analysts to detect malware, tampering, evasion, and early-stage ransomware indicators through Defender logs.

Next:  
➡️ **Section 5 — Task Scheduler Logs**
