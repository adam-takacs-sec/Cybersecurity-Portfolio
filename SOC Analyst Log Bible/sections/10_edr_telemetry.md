# **Section 10 — EDR Telemetry**
_Comprehensive SOC, DFIR, Threat Hunting & Detection Engineering Reference_

Endpoint Detection & Response (EDR) systems provide the deepest, most actionable telemetry for detecting modern attacks.

While Windows Security Logs & Sysmon show *events*,  
**EDR shows behavior**, including:

- process tree visualizations  
- command-line arguments  
- network callbacks  
- file modifications  
- memory injections  
- credential access  
- lateral movement techniques  
- malicious parent-child relationships  
- script execution  
- in-memory payloads (non-file-based attacks)  

This section documents the most critical EDR telemetry types and how SOC analysts interpret them.

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of EDR Telemetry](#-overview-of-edr-telemetry)
- [Process Trees](#-process-trees)
  - [Suspicious Parent-Child Chains](#-suspicious-parent-child-chains)
- [Command Line Indicators](#-command-line-indicators)
- [Network Telemetry](#-network-telemetry)
- [Memory & Injection Indicators](#-memory--injection-indicators)
- [Credential Access Indicators](#-credential-access-indicators)
- [Lateral Movement Indicators](#-lateral-movement-indicators)
- [File & Registry Telemetry](#-file--registry-telemetry)
- [Persistence Indicators](#-persistence-indicators)
- [Behavioral Detection Patterns](#-behavioral-detection-patterns)
- [EDR SOC Red Flags](#-edr-soc-red-flags)

---

# ---
# ## **Overview of EDR Telemetry**

EDRs like:
- Microsoft Defender for Endpoint (MDE)  
- CrowdStrike Falcon  
- SentinelOne  
- Carbon Black  
- Elastic EDR  
- Sophos Intercept X  

Collect behavioral signals that cannot be reliably seen anywhere else.

Examples:
- PowerShell spawning C2 traffic  
- Memory injection into LSASS  
- Tools like Rubeus, Mimikatz, SharpHound  
- Credential dumping  
- In-memory shellcode execution  
- Behavioral ransomware signatures  
- Living-off-the-Land binary (LOLBIN) abuse  

This is extremely high-quality telemetry.

---

# ---
# ## **Process Trees**

Process trees show *who executed what*.  
This is crucial for distinguishing legitimate admin behavior from malicious activity.

Example malicious tree:
```
WINWORD.EXE
  └── powershell.exe
        └── rundll32.exe
              └── http connection → 185.199.22.41:443
```

### **SOC Insight**
- Microsoft Word should never spawn PowerShell  
- Rundll32 should never talk to the internet  
- EDR would flag this as malicious chain behavior  

---

# ###
# ## **Suspicious Parent-Child Chains**

### **1️⃣ Office → PowerShell**  
Phishing / macro abuse.

### **2️⃣ Office → CMD → PowerShell**  
Obfuscation or staged execution.

### **3️⃣ Browser → certutil.exe**  
Malware downloading.

### **4️⃣ Service Host → powershell.exe**  
Service impersonation.

### **5️⃣ Explorer → rundll32.exe (odd DLL)**  
User clicked on disguised malware.

### **6️⃣ powershell.exe → regsvr32.exe → outbound traffic**  
Multi-stage attack.

### **7️⃣ cscript/wscript → powershell.exe**  
Malicious VBS/JS.

---

# ---
# ## **Command Line Indicators**

Attackers rely heavily on encoded or obfuscated commands.

Common malicious strings:

| Pattern | Meaning |
|--------|---------|
| `powershell -nop -w hidden` | stealth execution |
| `-enc` | Base64 encoded payload |
| `IEX (New-Object Net.WebClient)` | download + execute |
| `Invoke-Mimikatz` | credential dumping |
| `FromBase64String` | in-memory decoding |
| `Invoke-WebRequest` | external payload fetch |
| `System.Net.WebClient` | C2 communication |
| `Add-MpPreference` | Defender evasion |

### **EDR Decoding**
Unlike logs, many EDRs automatically decode Base64 PowerShell commands.  
This reveals full attacker intent.

---

# ---
# ## **Network Telemetry**

EDR captures:
- destination IP  
- destination port  
- process making the connection  
- protocol  
- byte counts  
- frequency (beaconing patterns)  

### **Red Flags**
- PowerShell contacting unknown IPs  
- Rundll32 making HTTPS requests  
- High-frequency beaconing every 10–30 seconds  
- Connections to:
  - VPS providers  
  - Bulletproof hosting  
  - Rare countries  

### **Example EDR Entry**
```
Process: powershell.exe
Destination: 45.79.221.14:443
Pattern: Beaconing every 15 seconds
```

This is *classic Cobalt Strike* behavior.

---

# ---
# ## **Memory & Injection Indicators**

EDR detects:
- process hollowing  
- remote thread creation  
- reflective DLL injection  
- shellcode injection  
- PE injection into legitimate processes  

### **Red Flag Destinations**
- lsass.exe  
- winlogon.exe  
- explorer.exe  
- svchost.exe  

### **Example**
```
Alert: Suspicious memory injection
Source: powershell.exe
Target: lsass.exe
Technique: CreateRemoteThread
```

### **MITRE**
- **T1055 — Process Injection**
- **T1003 — Credential Dumping**

---

# ---
# ## **Credential Access Indicators**

EDR identifies tools and behaviors tied to credential theft:

### **1️⃣ LSASS access**
```
OpenProcess → lsass.exe
ReadProcessMemory → lsass.exe
```

### **2️⃣ Known tooling**
- Mimikatz  
- Rubeus  
- HackTools  
- Cobra Mimikatz Kits  

### **3️⃣ SAM / SECURITY hive access**
```
reg save hklm\sam C:\Temp\sam.save
```

### **4️⃣ DPAPI abuse**
- decrypting browser passwords  
- decrypting credentials in memory  

### **5️⃣ Token theft**
- Pass-the-Token  
- Pass-the-Hash  
- Kerberos ticket replay  

### **MITRE**
- **T1003 — Credential Dumping**  
- **T1550 — Use of Stolen Credentials**

---

# ---
# ## **Lateral Movement Indicators**

### **1️⃣ Remote execution using:**
- WinRM  
- WMI  
- PsExec  
- SMB  
- RDP  

### **2️⃣ File dropped onto ADMIN$ or C$ shares**
```
C:\Windows\Temp\evil.exe copied via SMB
```

### **3️⃣ Explorer.exe spawning remote execution tools**

### **4️⃣ EDR sees new credentials in use**
- new logon tokens  
- unexpected admin access  

### **5️⃣ Credential use on multiple machines quickly**
Indicates worm-like spread of credentials.

---

# ---
# ## **File & Registry Telemetry**

EDR tracks:
- file creation/deletion  
- registry modifications  
- driver installation  
- service creation  

### **Red Flags**
- files written to:
  - `AppData`
  - `Temp`
  - `ProgramData`
  - `Public`  

- registry modifications to:
  - `Run` keys  
  - Defender configuration  
  - Services  

---

# ---
# ## **Persistence Indicators**

Attackers rely on EDR-visible persistence mechanisms:

### **1️⃣ Scheduled Tasks**
Hidden or modified tasks.

### **2️⃣ Registry Run Keys**
```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

### **3️⃣ WMI Event Consumers**

### **4️⃣ DLL Hijacking**

### **5️⃣ Service Creation**
Malicious services appearing as:
- updater  
- windows update  
- sysservice  
- maintenance  

### **6️⃣ Startup folder scripts**

---

# ---
# ## **Behavioral Detection Patterns**

### **1️⃣ Execution + Network + File Write**
Classic malware pattern:
```
powershell.exe → downloads payload → writes file → executes
```

### **2️⃣ Short-lived processes**
Malware often:
- spawns a process  
- does one action  
- exits  

### **3️⃣ Unusual parent processes**
Example:
```
winword.exe → cmd.exe → powershell.exe
```

### **4️⃣ Repeated failed authentications**
Credential theft attempts.

### **5️⃣ LSASS access without lsass-related parent**
Suspicious:
```
powershell.exe → lsass.exe
```

---

# ---
# ## **EDR SOC Red Flags**

### **High-Critical Indicators**
- PowerShell spawning external network traffic  
- Memory injection into lsass.exe  
- outbound connections every X seconds  
- encoded PowerShell commands  
- Rundll32 executing from odd locations  
- tampering with Defender or EDR services  
- access to SAM / SECURITY hives  
- remote execution tools  
- persistence in multiple places  

### **Very High-Critical Indicators**
- LSASS dump  
- credential theft tools detected  
- remote thread creation  
- in-memory execution of shellcode  
- ransomware behavioral patterns  
- encryption activity spikes  

### **Immediate Response Required**
- multiple machines showing beaconing  
- identity theft: same user from multiple countries  
- admin access suddenly granted  
- new service accounts created  
- EDR disabled or tampered  

---

# 🎯 **End of Section 10 — EDR Telemetry**
This final section covers all critical EDR behavioral signals used in modern SOC operations, threat hunting, and DFIR investigations.

With Section 10 completed, the **SOC Analyst Log Bible is now fully written**.

You have:
- Windows Logs  
- Sysmon  
- PowerShell  
- Defender  
- Scheduled Tasks  
- Linux Logs  
- Network Logs  
- Web Server Logs  
- Cloud Logs  
- EDR Telemetry  

