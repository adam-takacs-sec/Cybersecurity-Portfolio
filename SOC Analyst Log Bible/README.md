# 📘 SOC Analyst Log Bible  
### *The Ultimate Log Reference for Blue Team, DFIR, SOC, Threat Hunting & Detection Engineering*

A SOC analyst's work is 90% log interpretation, correlation, and analysis.  
This project is one of the **most complete and practical log references**, covering:

- Windows Event Logs (Security, System)  
- Sysmon (all important Event IDs with examples)  
- PowerShell Operational & ScriptBlock logs  
- Windows Defender & Task Scheduler  
- Linux logs (auth.log, syslog, auditd, systemd)  
- Firewall logs (Palo Alto, Fortinet, Cisco ASA)  
- VPN logs (AnyConnect, GlobalProtect)  
- DNS logging (server + endpoint)  
- Proxy & Web Gateway logs  
- DHCP logs (lateral movement detection)  
- IIS, Apache, NGINX logs  
- Cloud provider audit logs (Azure, AWS, GCP)  
- EDR telemetry (vendor-agnostic behavioral breakdown)  
- MITRE ATT&CK mapping for every event type  
- "Alert Relevance" scoring for SOC analysts  

This project brings **every important log source into one place**, explained clearly with examples and practical guidance.  
The goal: *never again have a moment of “I don’t know what this event means”* as a SOC analyst.

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
