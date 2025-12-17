# 🟥 Scenario 3 — Detection & Response in Action
**Attacker returns — SOC detects, contains, and stops the intrusion**

---

## 🧭 Scenario Context

Scenario 3 follows directly after Scenario 2.

Because the previous incident was not properly handled, the attacker retained valid credentials and decided to return.  
However, since Scenario 2, the organization introduced **active detection and response capabilities** in the SIEM.

This scenario demonstrates a realistic blue team success case:
> **The attacker gains access — but is quickly detected, contained, and forced to disengage.**

---

## 🟦 1. Environment Overview — Defensive State

The Windows Server endpoint remains externally accessible, but detection maturity has significantly improved.

### 🔐 Account & Authentication State

- Password complexity: **Enabled**
- Minimum password length: **10**
- Password history: **5**
- Maximum password age: **90 days**
- Account lockout policy: **Configured**
- Compromised account: **Still valid at scenario start**

---

## 📊 2. Logging & Detection Capabilities

### ✔ Active Telemetry

- Windows Security Event Logs
  - Successful logons (4624)
  - Failed logons (4625)
- NTLM authentication visibility
- Sysmon process creation events
- PowerShell execution visibility (process-level)

### ⚠ Known Limitation

- PowerShell command content is **not logged**
- Script Block Logging (Event ID 4104) not enabled

This reflects a **realistic partial visibility state**.

---

## 🟧 3. Attacker Re-Entry — Successful Authentication

Using previously compromised credentials, the attacker authenticated via RDP.

### Evidence

- Successful logon observed
- External attacker IP clearly visible
- Authentication alert elevated by correlation logic

📸 Screenshot:
- Wazuh Discover — Successful logon (original attacker IP)  
  ![Original IP](/Assets/Scenario_3/original_ip.png)

---

## 🟥 4. Post-Login Activity — Initial Reconnaissance

After gaining access, the attacker performed standard initial reconnaissance.

### Actions Performed

- Identity checks:
  - `whoami`
  - `hostname`
- System and environment enumeration:
  - `systeminfo`
  - `ipconfig /all`
  - `Get-LocalUser`

### Detection

- PowerShell process execution detected
- Process creation events generated (Event ID 4688)

⚠ Due to missing script block logging, exact command content was not available.

---

## 🟦 5. SOC Detection & Initial Assessment

The SOC identified a suspicious pattern:

- Successful authentication from an external IP
- Immediate PowerShell-based reconnaissance
- Indicators consistent with early-stage attacker activity

The incident was escalated for response.

---

## 🟥 6. SOC Response — Session Termination

As an immediate containment action, the SOC:

- Identified the active RDP session
- Terminated the session using a logoff action

### Result

- Interactive attacker access was interrupted
- No persistence mechanisms were established

---

## 🟥 7. SOC Containment — IP-Based Blocking

To prevent re-entry, the SOC applied:

- IP-based blocking against the attacker’s original source address

### Result

- RDP connections from the original IP were denied
- No additional successful logons occurred from that address

📸 Screenshot:
- IP block on port 3389 (RDP)
  ![IP Block port](/Assets/Scenario_3/ip_block_port.png)

- IP block by IP (178.164.153.115)
  ![IP Block IP](/Assets/Scenario_3/ip_block_ip.png)

---

## 🟧 8. Attacker Evasion Attempt — IP Change

The attacker attempted to bypass containment by:

- Changing external IP address (VPN-based evasion)
- Retrying RDP authentication using the same username

### Result

- Authentication failed
- Credential had already been reset by the SOC

📸 Screenshot:
- Wazuh Discover — Successful logon (original attacker IP)  
  ![Changed IP](/Assets/Scenario_3/changed_ip.png)

---

## 🔁 9. Post-Containment Activity

After containment:

- Additional failed authentication attempts were observed
- No successful logons occurred
- Attacker activity gradually stopped

This behavior indicates **opportunistic retry attempts**, not active compromise.

---

## 📊 10. Wazuh Dashboard — Incident Overview

The Wazuh dashboard provides a consolidated view of the entire incident lifecycle.

### What the Dashboard Shows

- Transition from successful authentication to containment
- Elevation of alert severity during attacker activity
- Clear disappearance of successful logons after SOC action
- Persistence of failed logons only

📸 Screenshots:
- Wazuh Dashboard — Incident overview (part 1)  
  ![Dashboard 1](/Assets/Scenario_3/dashboard_1.png)
- Wazuh Dashboard — Incident overview (part 2)  
  ![Dashboard 2](/Assets/Scenario_3/dashboard_2.png)

These views confirm that **SOC response directly altered the attack outcome**.

---

## 🧠 11. SOC Outcome & Lessons Learned

### What Worked

- Timely detection of suspicious authentication
- Correlation-based alerting
- Rapid session termination
- Effective IP-based containment
- Credential reset prevented re-entry

### Remaining Gaps

- Lack of PowerShell command-level visibility
- Script Block Logging not enabled during the incident

---

## 🟩 12. Scenario 3 Outcome

- Initial access: **Yes**
- Lateral movement: **No**
- Privilege escalation: **No**
- Persistence: **No**
- Data access/exfiltration: **No**

✅ **The attacker was detected early and successfully stopped.**

---

## 🔒 13. Lead-In to Hardening Phase

Following this incident, the organization proceeds with:

- Enhanced PowerShell logging
- Stricter authentication controls
- Reduced attack surface
- Mature SOC workflows

This marks the transition from **reactive detection** to **proactive defense**.

---

# ✔ End of Scenario 3

**Snapshots:**
- Windows → `scenario3_end_windows`
- Kali → `scenario3_end_kali`
