# 📘 SOC Analyst Log Bible
### *The Ultimate Log Reference for Blue Team, DFIR, SOC, Threat Hunting & Detection Engineering*

A SOC analyst munkájának 90%-a **logok értelmezéséből**, korrelálásából és elemzéséből áll.  
Ez a dokumentum a világ egyik **legteljesebb és leggyakorlatiasabb log-referenciája**, amely lefedi:

- Windows Event Logs (Security, System)
- Sysmon (minden fontos Event ID példával)
- PowerShell Operational & ScriptBlock logs
- Windows Defender & Task Scheduler
- Linux auth.log, syslog, auditd, systemd logs
- Firewall logs (Palo Alto, Fortinet, Cisco ASA)
- VPN logs (AnyConnect, GlobalProtect)
- DNS logging (server + endpoint)
- Proxy & Web Gateway logs
- DHCP logs (lateral movement detection)
- IIS, Apache, NGINX logs
- Cloud provider audit logs (Azure, AWS, GCP)
- EDR telemetry (vendor-agnostic breakdown)
- MITRE ATT&CK mapping minden eseményhez
- "Alert Relevance" szintek SOC analystek számára

Ez a projekt azért készült, hogy **minden fontos logforrást egy helyen**, érthetően, példákkal és magyarázatokkal együtt mutasson be.  
A cél: **SOC analystként soha többé ne legyen “nem tudom mi ez az event” pillanat.**

---

# 📚 Tartalomjegyzék

### 🔹 1. Windows Logok
- Security Log (Event IDs: 4624, 4625, 4672, 4688…)
- System Log (7045, 7036…)
- Application Log (fontosabb típusok)

### 🔹 2. Sysmon Logok
- Event ID-k részletes magyarázata
- MITRE TTP mapping
- Real-world attacker examples

### 🔹 3. PowerShell Logs
- ScriptBlock (4104)
- Module Logging (4103)
- Pipeline Execution (800)
- Malicious examples

### 🔹 4. Windows Defender Logs
- Malware detection (1116, 1117)
- Behavior blocking (5010)
- Config tampering (5007)

### 🔹 5. Task Scheduler Logs
- Persistence detection (106, 140, 200)

### 🔹 6. Linux Logs
- auth.log
- syslog
- auditd (EXECVE, SYSCALL, PATH…)
- sudo logs
- sshd logs
- persistence detection

### 🔹 7. Network Logs
- Firewall logs (Palo Alto, Fortinet, Cisco ASA)
- VPN logs (AnyConnect, GlobalProtect)
- DNS logs (server + endpoint)
- Proxy logs (Squid, Bluecoat)

### 🔹 8. Webserver Logs
- Apache
- NGINX
- IIS (cs-method, sc-status események)

### 🔹 9. Cloud Logs
- Azure AD Sign-in logs
- AWS CloudTrail
- GCP Audit Logs
- Identity & access patterns

### 🔹 10. EDR Telemetry
- Process tree & ancestry
- Behavior-based detections
- Network telemetry
- Lateral movement detection

---

# 🎯 Célja ennek a projektnek
Ez a dokumentum:

- **SOC analysteknek**  
- **Threat huntereknek**  
- **Blue teamereknek**  
- **Cyber defense tanulóknak**  
készült.

A cél az, hogy bárki:

✔ felismerje a támadásokat logokból  
✔ megértse a folyamatok összefüggését  
✔ lássa a MITRE ATT&CK relevanciát  
✔ hatékonyabban vizsgálja az eseményeket  
✔ könnyebben építsen detekciókat SIEM-ben

---

# 🧠 Felépítés

Minden logforrásnál szerepelni fog:

### • Event ID  
### • Mit jelent?  
### • Miért fontos a SOC számára?  
### • Melyik MITRE technikához tartozik?  
### • Real attacker example log  
### • Mit kell keresni benne?  
### • Hogyan néz ki SIEM-ben?  

Ez lesz az eddigi **legpraktikusabb és legkönnyebben használható** log-hivatkozási anyag.

---

# 🚀 Továbblépés
Nyisd meg a külön fejezeteket a `/sections` mappában.  
Ha új logforrásokat akarsz hozzáadni, PR-t is küldhetsz.

---

Készen állsz arra, hogy úgy értsd a logokat, mint egy valódi SOC analyst?

**Let’s hunt.** 🔍
