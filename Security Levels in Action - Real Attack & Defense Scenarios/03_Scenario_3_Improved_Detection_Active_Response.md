# 🟥 Scenario 3 — Detection & Response in Action  
**I returned as the attacker — the SOC detected, contained, and stopped the intrusion**

---

## 🧭 Scenario Context

This scenario follows directly after Scenario 2.

Because the previous incident was not properly handled, I retained valid credentials and decided to return.  
However, after Scenario 2, I had already introduced **active detection and response capabilities** into the SIEM environment.

This scenario demonstrates a realistic blue team success case:

> **I gained access — but was quickly detected, contained, and forced to disengage.**

---

## 🟦 1. Environment Overview — Defensive State

The Windows Server endpoint remained externally accessible, but detection maturity had significantly improved.

### 🔐 Account & Authentication State

- Password complexity: Enabled
- Minimum password length: 10
- Password history: 5
- Maximum password age: 90 days
- Account lockout policy: Configured
- Compromised account: Still valid at scenario start

This state represents a system that is **still reachable**, but no longer passively observed.

---

## 📊 2. Logging & Detection Capabilities

### ✔ Active Telemetry

At the start of this scenario, the following telemetry was actively collected and analyzed:

- Windows Security Event Logs
  - Successful logons (4624)
  - Failed logons (4625)
- NTLM authentication visibility
- Sysmon process creation events
- PowerShell execution visibility (process-level)

### ⚠ Known Limitation

- PowerShell command content was **not logged**
- Script Block Logging (Event ID 4104) was not enabled

This reflects a **realistic partial-visibility environment**, where detection exists but is not yet perfect.

---

## 🟧 3. Attacker Re-Entry — Successful Authentication

Using previously compromised credentials, I authenticated via RDP.

### Evidence Observed

- Successful logon event generated
- External attacker IP clearly visible
- Authentication alert elevated by correlation logic

Wazuh Discover — Successful logon from original attacker IP  
![Original IP](/Assets/Scenario_3/original_ip.png)

At this point, the system allowed access, but the activity did not go unnoticed.

---

## 🟥 4. Post-Login Activity — Initial Reconnaissance

After gaining access, I performed standard initial reconnaissance.

### Actions Performed

- Identity checks:
  - `whoami`
  - `hostname`
- System and environment enumeration:
  - `systeminfo`
  - `ipconfig /all`
  - `Get-LocalUser`

### Detection Outcome

- PowerShell process execution was detected
- Process creation events were generated (Event ID 4688)

Due to missing Script Block Logging, the **exact command content was not visible**, only the execution context.

---

## 🟦 5. SOC Detection & Initial Assessment

Based on the collected telemetry, I (acting as the SOC) identified a suspicious pattern:

- Successful authentication from an external IP
- Immediate PowerShell-based reconnaissance
- Activity consistent with early-stage attacker behavior

The incident was escalated for response.

---

## 🟥 6. SOC Response — Session Termination

As an immediate containment action, I:

- Identified the active RDP session
- Terminated the session using a logoff action

### Result

- Interactive attacker access was interrupted
- No persistence mechanisms were established
- No further commands could be executed in the active session

---

## 🟥 7. SOC Containment — IP-Based Blocking

To prevent re-entry from the same source, I applied:

- IP-based blocking against the attacker’s original external IP address

### Result

- RDP connections from the original IP were denied
- No additional successful logons occurred from that address

Firewall evidence:

- RDP port block (3389)  
  ![IP Block Port](/Assets/Scenario_3/ip_block_port.png)

- Explicit IP block rule  
  ![IP Block IP](/Assets/Scenario_3/ip_block_ip.png)

---

## 🟧 8. Attacker Evasion Attempt — IP Change

I then attempted to bypass containment by:

- Changing my external IP address using a VPN
- Retrying RDP authentication with the same username

### Result

- Authentication failed
- The credential had already been reset by the SOC

Wazuh Discover — Authentication attempt from changed IP  
![Changed IP](/Assets/Scenario_3/changed_ip.png)

This confirmed that containment actions were effective beyond simple IP blocking.

---

## 🔁 9. Post-Containment Activity

After containment:

- Additional failed authentication attempts were observed
- No successful logons occurred
- Attacker activity gradually stopped

This behavior indicates **opportunistic retry attempts**, not an active compromise.

---

## 📊 10. Wazuh Dashboard — Incident Overview

The Wazuh dashboard provided a consolidated view of the entire incident lifecycle.

### What I Observed on the Dashboard

- Clear transition from successful authentication to containment
- Elevated alert severity during attacker activity
- Disappearance of successful logons after response actions
- Persistence of failed logons only

Wazuh Dashboard — Incident overview (part 1)  
![Dashboard 1](/Assets/Scenario_3/dashboard_1.png)

Wazuh Dashboard — Incident overview (part 2)  
![Dashboard 2](/Assets/Scenario_3/dashboard_2.png)

These views confirm that **SOC actions directly altered the attack outcome**.

---

## 🧠 11. SOC Outcome & Lessons Learned

### What Worked

- Timely detection of suspicious authentication
- Correlation-based alerting
- Rapid session termination
- Effective IP-based containment
- Credential reset preventing re-entry

### Remaining Gaps

- Lack of PowerShell command-level visibility
- Script Block Logging not enabled during the incident

These gaps were identified for post-incident hardening.

---

## 🟩 12. Scenario 3 Outcome

- Initial access: Yes
- Lateral movement: No
- Privilege escalation: No
- Persistence: No
- Data access or exfiltration: No

**The attacker was detected early and successfully stopped.**

---

## 🔒 13. Lead-In to Hardening Phase

Following this incident, I proceeded with:

- Enhanced PowerShell logging
- Stricter authentication controls
- Reduced attack surface
- Mature SOC workflows

This marks the transition from **reactive detection** to **proactive defense**.
