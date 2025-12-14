# 🟩 Scenario 4 — Hardened Environment Stops the Attacker  
### *Attacker cannot even begin the attack chain — security controls and user awareness shut down every possible entry point.*

This scenario represents the final stage of the company's security maturity development.

By now, the following improvements have been implemented (thanks to incidents detected in Scenario 3):

- ✔ Strong password policies  
- ✔ MFA enabled for administrative access  
- ✔ RDP removed from public exposure  
- ✔ Network access segmentation  
- ✔ Strict firewall rules  
- ✔ User awareness training (no more phishing victims)  
- ✔ SOC playbooks created  
- ✔ Detection engineering fully tuned  
- ✔ Backdoor hunting & baseline monitoring enabled  
- ✔ Cloud firewall blocks unknown IPs  
- ✔ All old persistence artifacts removed  

The attacker tries multiple entry methods — **every single one fails**.

This scenario demonstrates a mature security posture where the attack cannot even start.

---

# 🟦 1. Environment Overview (After Scenario 3 Hardening)

The company applied all recommended security improvements:

## 🔐 1.1 Account & Password Security
- Password length: **min. 12 characters**
- Complexity: **enabled**
- Lockout threshold: **5 failed attempts**
- Lockout duration: **30 minutes**
- Impossible to password spray or brute force
- All administrative logins require **MFA**

---

## 🔐 1.2 Privileged Access Management
- No local admin users except `Administrator`
- All unnecessary accounts removed
- Privileged actions require strong authentication

---

## 🧱 1.3 Network Hardening
- ❌ **RDP fully disabled externally**
- ✔ Only accessible from internal VPN subnet  
- ✔ Firewall IP allowlist implemented  
- ✔ Cloud firewall blocks:
  - Every unknown IP  
  - Every port except 443/80 (web)  
  - RDP, WinRM, SMB blocked externally

---

## 👁️ 1.4 SIEM & SOC Improvements
- Custom dashboards in Wazuh  
- Alert correlation active  
- Behavioral detection enabled  
- Analyst playbooks implemented  
- SOC shifts monitor every 30 minutes  
- Automated blocking for:
  - Multiple failed logins  
  - Suspicious PowerShell activity  
  - Unknown admin accounts  
  - External RDP attempts  

---

## 🧠 1.5 User Awareness Improvements
- Staff trained to:
  - Recognize phishing  
  - Not click unknown links  
  - Identify spoofed emails  
  - Report suspicious messages immediately  
- HR, Finance and IT validated all mail workflows  
- The user from Scenario 3 (Anna) now refuses unknown login links

---

# 🟠 2. Before Starting Scenario 4

### ✔ Snapshot Windows Server  
`scenario4_start_windows`

### ✔ Snapshot Kali Linux  
`scenario4_start_kali`

Everything is expected to fail — this scenario documents **defensive success**, not attacker success.

---

# 🟥 3. Attacker Attempts to Re-Enter — All Methods Blocked

Below are the exact methods the attacker tries from Kali and what happens.

---

# 🟥 3.1 Attempt 1 — Reuse Old Backdoor Credentials

```bash
xfreerdp /v:<WINDOWS_IP> /u:backupadmin /p:Winter2024! /cert:ignore
```

### Expected:
❌ Firewall blocks connection (no response)  
❌ Account was deleted in Scenario 3  
❌ RDP not exposed externally  

**SIEM:**  
- No incoming RDP logs → connection dropped at perimeter  

---

# 🟥 3.2 Attempt 2 — OSINT Password Guessing (Scenario 2 Style)

```bash
hydra -l akovacs -P osint_passwords.txt rdp://<WINDOWS_IP>
```

### Expected:
❌ Connection refused  
❌ RDP port not reachable  
❌ Cloud firewall alert for blocked port scans  

**SIEM:**  
- Detection of port scanning attempt  
- Incident escalated automatically  

---

# 🟥 3.3 Attempt 3 — Try WinRM Backdoor Port (5987 from Scenario 2)

```bash
evil-winrm -i <WINDOWS_IP> -u backupadmin -p Winter2024! -P 5987
```

### Expected:
❌ Port 5987 removed in Scenario 3  
❌ Firewall blocks ALL traffic on non-approved ports  

---

# 🟥 3.4 Attempt 4 — SMB/Network Recon

```bash
nmap -sV -p 445 <WINDOWS_IP>
```

### Expected:
❌ “host filtered”  
❌ Cloud firewall blocks the packet  

**SIEM:**  
- Security alerts: external SMB scan blocked  

---

# 🟥 3.5 Attempt 5 — Phishing Attempt

Attacker sends a fake “IT Security Password Reset” email to Anna.

### Expected User Behavior:
✔ Recognizes it as phishing  
✔ Does NOT click  
✔ Reports email to SOC  
✔ SOC tags the phishing site, blocks IP automatically  

**SOC Reaction:**  
- Creates incident ticket  
- Adds indicator to threat blacklist  
- Sends awareness follow-up  

This demonstrates the human improvement since previous scenarios.

---

# 🟥 3.6 Attempt 6 — Try Zero-Interaction Exploits

Attacker tries generic exploit scanning:

```bash
nmap --script vuln <WINDOWS_IP>
```

### Expected:
❌ All ports blocked  
❌ Zero attack surface exposed  
❌ SIEM signatures detect scanning behavior  

---

# 🟩 4. SOC Verification — The System is Secure

SOC confirms:

- No unauthorized users created  
- No login attempts succeeded  
- No persistence found  
- All firewall policies active  
- No privilege escalations  
- All Phishing attempts reported  
- No unusual outbound traffic  

Windows remains **fully uncompromised**.

---

# 🟦 5. Lessons Learned

### ✔ Proper configuration beats most attacks  
### ✔ External RDP is NEVER safe  
### ✔ MFA drastically reduces credential attack success  
### ✔ User awareness training is critical  
### ✔ SIEM visibility allows SOC to react BEFORE compromise  
### ✔ Perimeter firewalls must implement geo/IP filtering  
### ✔ Attackers fail when they cannot start the chain  

This scenario demonstrates the organization reaching a **mature, resilient security posture**.

---

# ✔ End of Scenario 4  
**Take snapshots:**

- Windows → `scenario4_end_windows`  
- Kali → `scenario4_end_kali`

