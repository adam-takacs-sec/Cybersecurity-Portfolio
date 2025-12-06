# **Section 2 — Sysmon Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

Sysmon (System Monitor) is one of the most powerful endpoint telemetry tools available.  
It provides deep visibility into process creation, network activity, registry changes,
file modifications, and many stealthy attacker techniques that Windows Security Logs
do not capture.

This section breaks down every important Sysmon Event ID used in real-world threat hunting,
detection engineering, and incident response.

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Sysmon](#-overview-of-sysmon)
- [Sysmon Configuration Importance](#-sysmon-configuration-importance)
- [Event ID 1 — Process Creation](#-event-id-1--process-creation)
- [Event ID 2 — File Creation Time Changed](#-event-id-2--file-creation-time-changed)
- [Event ID 3 — Network Connection](#-event-id-3--network-connection)
- [Event ID 7 — Image Loaded](#-event-id-7--image-loaded)
- [Event ID 8 — CreateRemoteThread](#-event-id-8--createremotethread)
- [Event ID 10 — Process Access](#-event-id-10--process-access)
- [Event ID 11 — File Created](#-event-id-11--file-created)
- [Event ID 12–13–14 — Registry Events](#-event-id-121314--registry-events)
- [Event ID 22 — DNS Query](#-event-id-22--dns-query)
- [Event ID 23 — File Deleted](#-event-id-23--file-deleted)
- [Event ID 25 — Process Tampering](#-event-id-25--process-tampering)

---

# ---
# ## **Overview of Sysmon**
Sysmon extends Windows logging with high-fidelity telemetry needed for detecting:
- malware execution  
- exploitation & post-exploitation  
- credential theft techniques  
- persistence mechanisms  
- lateral movement  
- C2 (command and control)  
- file system & registry manipulation  

Sysmon must be configured via **sysmon.xml**.  
The quality of logs depends entirely on the configuration.

Recommended configs:
- SwiftOnSecurity  
- Olaf Hartong's Sysmon Modular (enterprise-grade)  

---

# ---
# ## **Sysmon Configuration Importance**
A good config:
- Whitelists noise  
- Captures attacker behavior  
- Provides clear, SIEM-friendly events  

A bad config:
- Produces millions of useless logs  
- Overwhelms SIEM ingestion  
- Hides real attacker signals  

---

# ---
# ## **Event ID 1 — Process Creation**
### **Meaning**
A new process started, including:
- full image path  
- command line  
- parent process  
- hash  
- integrity level  

### **Why It Matters**
This is **the single most important Sysmon event**.

Detects:
- malware execution  
- phishing payloads  
- LOLBins  
- PowerShell abuse  
- encoded commands  
- command line obfuscation  
- post-exploitation frameworks  

### **MITRE ATT&CK**
- **T1059 — Command Execution**  
- **T1204 — User Execution**  
- **T1106 — Native API Execution**

### **Real Attacker Example**
```
EventID: 1
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
CommandLine: powershell -nop -w hidden -enc SQBtAHAAbwByAHQALA...
ParentImage: C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE
Hashes: MD5=8AD1FA29A1...
```

### **Detection Notes**
Flag:
- Base64 commands (`-enc`)  
- LOLBins:  
  - rundll32  
  - regsvr32  
  - mshta  
  - wscript/cscript  
- Office spawning PowerShell or cmd  
- Browser spawning certutil  
- Unexpected binaries in Temp/AppData  

### **SIEM Query (KQL)**
```kql
Sysmon
| where EventID == 1
| where ParentImage has_any ("winword.exe","excel.exe","outlook.exe")
| where Image has_any ("powershell.exe","cmd.exe","wscript.exe","mshta.exe")
```

---

# ---
# ## **Event ID 2 — File Creation Time Changed**
### **Meaning**
File creation timestamp modified (anti-forensics behavior).

### **Why It Matters**
Attackers use timestamp modification to:
- evade timeline forensics  
- hide malware deployment time  
- imitate system files  

### **MITRE ATT&CK**
- **T1070 — Indicator Removal**

### **Red Flags**
- Timestamps altered shortly after malware drop  
- Modifications in system folders  

---

# ---
# ## **Event ID 3 — Network Connection**
### **Meaning**
A process established an outbound network connection.

### **Why It Matters**
Essential for:
- C2 detection  
- identifying malware beaconing  
- spotting internal lateral movement  
- mapping persistence callbacks  

### **Key Fields**
- Source IP  
- Destination IP  
- Destination Port  
- Image  
- User  

### **MITRE ATT&CK**
- **T1071 — C2 Channels**  
- **T1021 — Remote Services**

### **Real Attacker Example**
```
EventID: 3
Image: powershell.exe
DestinationIp: 45.76.112.51
DestinationPort: 443
Initiated: true
```

### **Detection Notes**
Flag connections from:
- PowerShell  
- rundll32  
- regsvr32  
- mshta  
- wscript  
- unknown unsigned executables  

Red flags:
- Outbound 443 to VPS IPs  
- Frequent regular-interval beacons  
- Connections to IP-only hosts  

### **SIEM Detection**
```kql
Sysmon
| where EventID == 3
| where Image has "powershell.exe"
| project TimeGenerated, Image, DestinationIp, DestinationPort
```

---

# ---
# ## **Event ID 7 — Image Loaded**
### **Meaning**
A process loaded a DLL.

### **Why It Matters**
Used to detect:
- DLL sideloading  
- malicious DLL injection  
- unsigned DLL usage  
- persistence techniques  

### **MITRE ATT&CK**
- **T1574 — DLL Search Order Hijacking**

### **Red Flags**
- DLLs loaded from:
  - Temp  
  - AppData  
  - Public  
- Unsigned or suspicious DLLs  

### **Real Attacker Example**
```
ImageLoaded: C:\Users\Public\mscoree.dll
Signed: False
```

---

# ---
# ## **Event ID 8 — CreateRemoteThread**
### **Meaning**
A process created a thread in another process.

### **Why It Matters**
High-fidelity signal for:
- **process injection**
- malware loading into legitimate processes  
- credential harvesting tools  
- Cobalt Strike & similar frameworks  

### **MITRE ATT&CK**
- **T1055 — Process Injection**

### **Red Flags**
- Source: powershell.exe → Target: lsass.exe  
- Browser → Explorer.exe  
- Unknown unsigned executable → any system process  

---

# ---
# ## **Event ID 10 — Process Access**
### **Meaning**
A process attempted to access another process's memory.

### **Why It Matters**
This detects:
- Mimikatz  
- LSASS dumping  
- Credential harvesting  
- Token theft  
- Privilege escalation  

### **MITRE ATT&CK**
- **T1003 — Credential Dumping**
- **T1055 — Injection**

### **Real Attacker Example**
```
EventID: 10
SourceImage: powershell.exe
TargetImage: lsass.exe
GrantedAccess: 0x1010
```

### **Detection Notes**
Flag ANY non-system process accessing lsass.exe.

---

# ---
# ## **Event ID 11 — File Created**
### **Meaning**
A file was created.

### **Why It Matters**
Detects:
- malware dropper behavior  
- persistence artifacts  
- file-based payloads  
- droppers used by phishing attacks  

### **Red Flags**
- Executables created in:
  - Temp  
  - AppData  
  - Public  
- Script files:
  - .ps1  
  - .vbs  
  - .js  
  - .hta  

---

# ---
# ## **Event ID 12–13–14 — Registry Events**
### **Meaning**
Monitors registry:
- create  
- modify  
- delete  

### **Why It Matters**
Critical for detecting:
- persistence (Run keys, Services)  
- UAC bypass  
- Defender tampering  
- malware configuration changes  

### **MITRE ATT&CK**
- **T1112 — Registry Modification**
- **T1547 — Boot or Logon Autostart Execution**

### **Red Flags**
- Modifications to:
  - `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
  - `HKLM\SYSTEM\CurrentControlSet\Services\`  
  - `HKLM\Software\Policies\Microsoft\Windows Defender`  

---

# ---
# ## **Event ID 22 — DNS Query**
### **Meaning**
Process issued a DNS request.

### **Why It Matters**
Perfect for detecting:
- DNS-based C2  
- malware beaconing  
- DGA (Domain Generation Algorithm) domains  
- DNS tunneling  

### **MITRE ATT&CK**
- **T1071.004 — DNS C2**

### **Real Attacker Example**
```
Image: powershell.exe
QueryName: a9f3jd9ajdlfj9ajdf-malware.xyz
```

### **Detection Notes**
Flag:
- Random subdomains (DGA)  
- High-frequency requests to the same domain  
- TXT records with large payloads  

---

# ---
# ## **Event ID 23 — File Deleted**
### **Meaning**
File was removed from the system.

### **Why It Matters**
Detects:
- malware self-delete  
- anti-forensics  
- scripts cleaning traces  
- ransomware wiping initial payloads  

### **Red Flags**
- Deletions in Temp shortly after execution  
- Droppers removing themselves  
- System directories being wiped  

---

# ---
# ## **Event ID 25 — Process Tampering**
### **Meaning**
Windows process security was modified (e.g., AMSI bypass).

### **Why It Matters**
Detects:
- AMSI bypass  
- PatchGuard bypass  
- Obfuscation  
- stealthy malware EDR avoidance behavior  

### **MITRE ATT&CK**
- **T1562 — Defense Evasion**

---

# 🎯 **End of Section 2**
This section provides everything needed for Sysmon-based SOC detection, DFIR investigations, and detection engineering.

Next:  
➡️ **Section 3 — PowerShell Logs**
