# **Section 5 — Task Scheduler Logs**
_Comprehensive SOC, DFIR & Persistence Detection Reference_

The Windows Task Scheduler is one of the **most common persistence mechanisms** used by attackers.  
It is trivial to abuse, survives reboots, and blends into normal system behavior if not monitored properly.

This section documents all key Task Scheduler event IDs relevant to SOC analysts and detection engineering, along with attacker examples and detection logic.

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Task Scheduler Logging](#overview-of-task-scheduler-logging)
- [Event ID 106 — Task Registered](#event-id-106--task-registered)
- [Event ID 140 — Task Updated](#event-id-140--task-updated)
- [Event ID 200 — Task Executed](#event-id-200--task-executed)
- [Common Attacker Techniques](#common-attacker-techniques)
- [Persistence Red Flags](#persistence-red-flags)
- [SIEM Detection Queries](#siem-detection-queries)

---

# ---
# ## **Overview of Task Scheduler Logging**

Task Scheduler logs are stored here:

```
Applications and Services Logs/
  Microsoft/
    Windows/
      TaskScheduler/
        Operational
```

Attackers use scheduled tasks for:
- persistence  
- execution after reboot  
- delayed execution (timed payload)  
- stealthy malware loading  
- privilege escalation (running as SYSTEM)  

Typical malicious behavior:
- Creating hidden tasks  
- Dropping executables into Temp/AppData  
- Triggering PowerShell payloads  
- Running commands from suspicious paths  

---

# ---
# ## **Event ID 106 — Task Registered**
### **Meaning**
A scheduled task was created (registered) on the system.

### **Why It Matters**
This is one of the top persistence indicators.

Attackers often register tasks to:
- start malware on boot  
- periodically download payloads  
- ensure access even after reboot  
- run commands silently as SYSTEM  

### **MITRE ATT&CK**
- **T1053.005 — Scheduled Task / Job**
- **T1543 — Create or Modify System Process**

### **Real Attacker Example**
```
EventID: 106
TaskName: \Microsoft\Windows\Update\update_service
UserContext: SYSTEM
Author: CORP\john.doe
Action: powershell.exe -nop -w hidden -enc AAAABBBBCCC...
```

### **Fields to Monitor**
- **TaskName** → suspicious names: “update”, “service”, “winsvc”, “googleupdate”, “officeupdate”  
- **Action** → look for PowerShell, cmd, rundll32, regsvr32  
- **Author** → unusual users creating tasks  
- **UserContext** → SYSTEM tasks created by non-admins  

### **Detection Notes**
Flag:
- Tasks created under `\Microsoft\Windows\*` by regular users  
- Executables from:
  - `%AppData%`
  - `%Temp%`
  - `C:\Users\Public\`  
- Base64 encoded commands (`-enc`)  

---

# ---
# ## **Event ID 140 — Task Updated**
### **Meaning**
A task was modified—sometimes more suspicious than creation.

### **Why It Matters**
Attackers may:
- modify an existing legitimate task  
- replace a benign action with malicious execution  
- update triggers to execute malware  

This is a **stealth persistence technique** because modifying existing tasks draws less attention than creating new ones.

### **MITRE ATT&CK**
- **T1053.005 — Modify Scheduled Task**
- **T1547 — Boot or Logon Autostart Execution**

### **Real Example**
```
EventID: 140
TaskName: \Microsoft\Windows\Defrag\ScheduledDefrag
Author: CORP\mary
NewAction: C:\Users\Public\svchost.exe
```

### **Red Flags**
- System maintenance tasks executing:
  - PowerShell scripts  
  - non-Microsoft executables  
  - binaries from AppData/Public  

- Triggers changed to odd schedules  
- Modifications by non-admin accounts  

---

# ---
# ## **Event ID 200 — Task Executed**
### **Meaning**
A scheduled task was triggered and executed.

### **Why It Matters**
Essential for:
- tracking persistence activation  
- correlating with malware execution  
- identifying beaconing patterns  
- discovering tasks that run every few minutes  

### **Real Example**
```
EventID: 200
TaskName: \Microsoft\Windows\Update\update_service
Action: powershell.exe -nop -w hidden -enc ...
```

### **What to Look For**
- Task execution shortly after system startup  
- High-frequency execution (every minute)  
- Execution at unusual hours  
- Repeated task executions across many hosts (worm-like spread)  

---

# ---
# ## **Common Attacker Techniques**

### **1️⃣ Create hidden persistence tasks**
```
schtasks /create /tn "WindowsUpdate" /tr "script.ps1" /sc MINUTE /mo 10 /ru SYSTEM
```

### **2️⃣ Modify legitimate tasks to load malware**
```
schtasks /change /tn "Defrag" /tr "C:\Users\Public\malware.exe"
```

### **3️⃣ Tasks launching encoded PowerShell**
```
powershell -nop -w hidden -enc AAAABBBBCCC...
```

### **4️⃣ Using `%AppData%` or `%Temp%` as execution folder**
Example:
```
C:\Users\admin\AppData\Local\Temp\update.exe
```

### **5️⃣ Using SYSTEM context to escalate privileges**

---

# ---
# ## **Persistence Red Flags**

### **Suspicious Task Names**
- update  
- updater  
- sysupdate  
- maintenance  
- service  
- googleupdate  
- officeupdate  

### **Suspicious Paths**
- `%AppData%\*.exe`
- `%Temp%\*.exe`
- `C:\Users\Public\*.exe`
- `C:\ProgramData\*.dll`

### **Suspicious Actions**
- PowerShell with `-nop -w hidden`  
- Encoded commands  
- Rundll32 executing non-Microsoft DLL  
- MSHTA loading remote HTA files  

### **Suspicious Timing**
- Runs every 1 minute  
- Runs at system idle  
- Runs at logon for all users  

---

# ---
# ## **SIEM Detection Queries**

### **KQL — Detect Suspicious Task Creation**
```kql
TaskScheduler
| where EventID == 106
| where Action contains_any ("powershell","cmd.exe","rundll32","mshta")
```

### **KQL — Detect Base64 Commands in Scheduled Tasks**
```kql
TaskScheduler
| where EventID == 106
| where Action contains "-enc"
```

### **KQL — Detect Tasks Created by Non-Admin Accounts**
```kql
TaskScheduler
| where EventID == 106
| where UserContext !contains "SYSTEM"
| where Author !contains_any ("Administrator","SYSTEM","TrustedInstaller")
```

### **KQL — Detect Modified Legitimate Tasks**
```kql
TaskScheduler
| where EventID == 140
| where NewAction contains_any ("AppData","Temp","Public")
```

### **KQL — Track Execution of Suspicious Tasks**
```kql
TaskScheduler
| where EventID == 200
| where Action contains_any ("powershell","cmd.exe","mshta","rundll32")
```

---

# 🎯 **End of Section 5**
This section provides all necessary detection logic for identifying persistence mechanisms, malicious scheduled tasks, and post-exploitation automation via Windows Task Scheduler.

Next:  
➡️ **Section 6 — Linux Logs**
