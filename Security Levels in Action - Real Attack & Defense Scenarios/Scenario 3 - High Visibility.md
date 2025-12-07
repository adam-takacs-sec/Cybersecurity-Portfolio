# 🟨 Scenario 3 — Backdoor Reuse, SOC Detection, Attacker Blocked  
### *Attacker returns through the hidden persistence placed in Scenario 2 — but this time, the SOC reacts and stops the breach.*

In this scenario:

- The attacker reuses the **backdoor account** created in Scenario 2  
- The company's SIEM is now well-tuned and has a **custom dashboard**  
- Analysts finally know what suspicious activity looks like  
- The intrusion is detected early  
- SOC reacts: **blocks IP**, **removes user**, **kills sessions**, **deletes persistence**  
- Attacker attempts fallback entry methods — **all fail**

This scenario demonstrates a huge defensive skill improvement.

---

# 🟦 1. Environment Overview (After Scenario 2 Improvements)

The company has improved significantly since Scenario 2:

## ✔ Better SIEM Tuning  
- Custom dashboard  
- MITRE-tagged alerts  
- Priority alerts for:
  - New admin users  
  - Scheduled task creation  
  - Firewall modification  
  - PowerShell ScriptBlock abuse  
  - RDP logons from unknown IPs  
  - Log clearing attempts

## ✔ SOC Analyst Now Actively Monitors Alerts  
- Starting shift dashboard review  
- Analysts respond within minutes, not hours  
- They know what the attacker did previously (Scenario 2 artifacts)

## ❌ But Some Technical Weaknesses Still Exist  
- RDP is still enabled  
- No MFA  
- No IP allowlist  
- Firewall rules not fully hardened  

This makes the environment **detectable but not yet impenetrable**.

---

# 🟠 2. Before Starting Scenario 3

### ✔ Snapshot Windows Server  
Name: `scenario3_start_windows`

### ✔ Snapshot Kali Linux  
Name: `scenario3_start_kali`

### ✔ Confirm the persistence from Scenario 2 exists
Inside Windows (for your verification only):

```powershell
net user backupadmin
schtasks /query | findstr SystemBackupTask
netsh advfirewall firewall show rule name="Backup WinRM"
```

You will not fix anything — this is the SOC’s job later.

---

# 🟧 3. Initial Access — Reusing the Backdoor Admin

The attacker returns days/weeks later and attempts logging in using the backdoor user:

```bash
xfreerdp /v:<WINDOWS_IP> /u:backupadmin /p:Winter2024! /cert:ignore
```

### Expected:
- Login success (LogonType 10 - RDP)
- SIEM immediately alerts:
  - **4624** admin login  
  - **New/unused account login**  
  - **Source IP never seen before**  
  - **High-risk account authentication**

**Screenshot:**  
➡️ Windows desktop after login  
➡️ SIEM → RDP admin login alert firing immediately

---

# 🟦 4. SOC Detection — Real-Time Response Begins

Within **1–3 minutes**, the SOC analyst sees:

### 🔥 Critical alerts:
- Login from unknown external IP  
- Privileged account used (4720/4732 history visible)  
- Previously unused user logging in  
- PowerShell events (ScriptBlock 4104)  
- Sysmon Process Creation (ID 1)

### 🕵️ Analyst flags session as suspicious  
They escalate the incident → start containment steps.

---

# 🟥 5. On-System Attacker Actions (Attempted but Detected Quickly)

The attacker tries to repeat initial recon:

```powershell
whoami
hostname
ipconfig /all
Get-LocalUser
```

SIEM immediately lists:

- Sysmon ID 1 (powershell.exe)  
- ScriptBlock 4104  
- RDP session metadata  
- Suspicious user behavior (“backupadmin” accessing server resources)

**Screenshot:**  
➡️ SIEM showing real-time process creation logs

---

# 🟪 6. SOC Blocks the Intrusion

## 6.1 Analyst Forces Session Termination

The SOC manually kills the RDP session:

```powershell
logoff <session_ID>
```

OR through Task Manager.

The attacker sees:

❌ **RDP session abruptly closed**

---

## 6.2 SOC Disables the Backdoor Account

```powershell
net user backupadmin /active:no
```

## 6.3 SOC Removes the Backdoor User Entirely

```powershell
net user backupadmin /delete
```

SIEM logs:
- 4726 — User account deleted  
- 4733 — User removed from Admin group

---

## 6.4 SOC Deletes the Attacker’s Scheduled Task

```powershell
schtasks /delete /tn "SystemBackupTask" /f
```

SIEM logs:
- 4699 — Scheduled task deleted

---

## 6.5 SOC Removes Firewall Backdoor Rule

```powershell
netsh advfirewall firewall delete rule name="Backup WinRM"
```

SIEM logs:
- Firewall rule modification

---

## 6.6 SOC Blocks Attacker IP at Firewall Level

Edge firewall / Cloud firewall:

```bash
block <ATTACKER_PUBLIC_IP>
```

Attacker loses ability to reconnect.

---

# 🟥 7. Attacker Attempts Fallback Methods (All Fail)

After being kicked out, the attacker tries different approaches:

---

## 7.1 Attempt RDP Login Again

```bash
xfreerdp /v:<WINDOWS_IP> /u:backupadmin /p:Winter2024!
```

**Expected:**  
❌ Account disabled or deleted

---

## 7.2 Attempt Using Original OSINT Password of Scenario 2 User

```bash
xfreerdp /v:<WINDOWS_IP> /u:<OLD_USER> /p:<OSINT_PASSWORD>
```

**Expected:**  
❌ Password changed after SOC containment

---

## 7.3 Attempt WinRM Backdoor Port (5987)

```bash
evil-winrm -i <WINDOWS_IP> -u backupadmin -p Winter2024! -P 5987
```

**Expected:**  
❌ Port closed  
❌ User removed  
❌ Connection refused

---

## 7.4 Attempt SMB / Lateral Entry

```bash
smbclient -L //<WINDOWS_IP> -U backupadmin
```

**Expected:**  
❌ Authentication failure  
❌ IP blocked upstream

---

# 🚫 Attacker cannot regain access.  
This is the first scenario where defense **beats** offense.

---

# 🟩 8. SOC Post-Incident Hardening

After containment, the SOC applies:

- Disable RDP exposure  
- Enforce MFA  
- Add IP allowlisting  
- Enable account lockout policy  
- Create SIEM rules for:
  - User creation  
  - Admin group changes  
  - Firewall modifications  
  - Scheduled tasks  
  - RDP brute-force detection  
- Add playbooks for:
  - Privileged account misuse  
  - Lateral movement investigation  
  - PowerShell detection  

This prepares the environment for **Scenario 4**, where the attacker fails before even entering.

---

# 🟦 9. Lessons Learned

### ✔ SOC reaction time is crucial  
### ✔ Persistence must always be checked after compromise  
### ✔ Every new admin user must trigger an alert  
### ✔ Firewall rule changes must be monitored  
### ✔ Credential-based attacks must be reviewed over time  
### ✔ External IP anomaly detection is essential  

The company is now **much stronger** than in Scenarios 1–2.

---

# ✔ End of Scenario 3  
**Create snapshots:**

- Windows → `scenario3_end_windows`  
- Kali → `scenario3_end_kali`
