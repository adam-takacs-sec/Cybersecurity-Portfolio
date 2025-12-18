# 📘 SOC Analyst Log Bible  
### *A Practical Log Reference for SOC, Blue Team, DFIR & Detection Engineering*

SOC analyst work relies heavily on understanding logs, correlating events, and interpreting activity across multiple systems.

This project is a hands-on log reference built around real SOC investigation workflows, not theoretical descriptions.

It brings together key log sources and explains what they actually tell you during an investigation — helping turn raw events into actionable understanding.

---

# 📚 **Table of Contents**

### 🔹 **1. Windows Logs**
- Security Log (4624, 4625, 4672, 4688…)  
- System Log (7045, 7036…)  
- Application Log (important types)

### 🔹 **2. Sysmon Logs**
- Detailed breakdown of Event IDs  
- MITRE TTP mapping  
- Real-world attacker examples  

### 🔹 **3. PowerShell Logs**
- ScriptBlock (4104)  
- Module Logging (4103)  
- Pipeline Execution (800)  
- Malicious patterns and examples  

### 🔹 **4. Windows Defender Logs**
- Malware detection (1116, 1117)  
- Behavior blocking (5010)  
- Config tampering (5007)

### 🔹 **5. Task Scheduler Logs**
- Persistence detection (106, 140, 200)

### 🔹 **6. Linux Logs**
- auth.log  
- syslog  
- auditd (EXECVE, SYSCALL, PATH…)  
- sudo events  
- sshd logs  
- persistence detection  

### 🔹 **7. Network Logs**
- Firewalls (Palo Alto, Fortinet, Cisco ASA)  
- VPN logs (AnyConnect, GlobalProtect)  
- DNS logs (server + endpoint)  
- Proxy logs (Squid, BlueCoat)

### 🔹 **8. Web Server Logs**
- Apache  
- NGINX  
- IIS (cs-method, sc-status, execution analysis)

### 🔹 **9. Cloud Logs**
- Azure AD Sign-in logs  
- AWS CloudTrail  
- GCP Audit Logs  
- Identity & access behavior patterns  

### 🔹 **10. EDR Telemetry**
- Process tree & ancestry  
- Behavioral detections  
- Network telemetry  
- Lateral movement detection  

---

# 🎯 **Purpose of This Project**

This document is designed for:

- SOC Analysts  
- Threat Hunters  
- Blue Team Engineers  
- Cyber Defense Students  

Its goal is to empower anyone to:

✔ Identify attacker behavior from logs  
✔ Understand relationships between events  
✔ Map logs to MITRE ATT&CK  
✔ Perform more effective investigations  
✔ Build stronger SIEM detections  

---

# 🧠 **Structure**

Every log source includes:

- **Event ID**  
- **What it means**  
- **Why it matters for SOC**  
- **MITRE ATT&CK mapping**  
- **Real attacker example log**  
- **What to look for**  
- **How it appears in SIEM queries**

This is designed to be the **most practical and easy-to-use log reference** available.
