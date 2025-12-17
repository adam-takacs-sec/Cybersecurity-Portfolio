# 🟧 Scenario 2 — Improved Visibility, Weak Response
Attacker gains access again — visibility improves, but no action is taken

---

## 🧭 Scenario Context

This scenario continues directly after Scenario 1.

Following the initial compromise, the organization implemented partial security improvements:
- Centralized logging was introduced
- A SIEM platform (Wazuh) was deployed
- Windows Security Event logs are actively ingested
- Sysmon was installed for enhanced telemetry
- Basic password policy improvements were applied

However, no SOC processes, alerting logic, or response workflows were introduced.

This scenario demonstrates a common real-world security failure:
The attacker is clearly visible — but ignored.

---

## 🟦 1. Environment Overview — Post Scenario 1 State

The Windows Server endpoint remains publicly accessible and operates as a workstation-style endpoint.

### Password & Account Security (Partially Improved)

- Password complexity enabled
- Minimum password length: 10
- Password history: 5
- Maximum password age: 90 days
- Account lockout policy: Disabled
- MFA: Not implemented

After the first incident, a real employee user changed their password.
While stronger than default credentials, the password was still guessable using OSINT-derived personal and contextual information.

---

## 📊 2. Logging & Visibility — Improved but Incomplete

### Advanced Audit Policy Configuration

The following audit categories were enabled:

- Logon / Logoff
  - Successful logons
  - Failed logons
- Account Logon
  - Credential validation
- Account Management
  - User and group changes
- Detailed Tracking
  - Process creation

These changes significantly increased authentication visibility.

---

## 👁️ 3. Sysmon Deployment

Sysmon was installed using a community-recommended baseline configuration.

Active telemetry includes:
- Process creation
- Network connections
- File creation
- Registry modifications

Sysmon events are forwarded to the SIEM via the Wazuh agent.

---

## 📡 4. SIEM State (Wazuh)

- Windows Security Event logs are collected
- NTLM authentication failures are visible
- RDP logons are visible
- Limited process execution telemetry is available

No alert rules, thresholds, or SOC playbooks are configured.

---

## 🧱 5. Network Exposure (Unchanged)

- RDP (3389) publicly exposed
- No IP allowlisting
- No rate limiting
- No brute-force detection rules
- No account lockout enforcement

---

## 🟧 6. Initial Access — OSINT-Based Credential Attack

### Attacker Preparation

Using information gathered during the OSINT pre-stage, the attacker prepared:
- A custom username list based on corporate naming conventions
- A targeted password list derived from personal and contextual information

Artifacts used:
- usernames_osint.txt
- passwords_osint.txt

[SCREENSHOT PLACEHOLDER]
Kali Linux — OSINT-based username and password lists
![Credidential List](/Assets/Scenario_2/credidential_list.png)

---

## 🔴 7. Authentication Attack Observed in SIEM

The attacker initiated repeated RDP authentication attempts against the exposed endpoint.

SIEM evidence confirms:
- Event ID 4625 (failed logon)
- NTLM authentication failures
- Status code 0xC000006A (wrong password)
- Repeated attempts against valid usernames
- High-volume authentication noise from a single source IP

This confirms:
- Valid usernames were discovered
- Password guessing was actively occurring
- No defensive controls interrupted the attack

[SCREENSHOT PLACEHOLDER]
Wazuh Discover — Failed authentication attempts
![4625](/Assets/Scenario_2/4625.png)

---

## 🟢 8. Successful Authentication

After multiple failed attempts, the attacker successfully authenticated using OSINT-derived credentials.

Observed behavior:
- Successful RDP logon (Event ID 4624)
- Multiple successful logon events due to RDP reconnect behavior
- Same source IP as the failed attempts

No alert was generated.
No SOC investigation followed.

[SCREENSHOT PLACEHOLDER]
Wazuh Discover — Successful NTLM logon
![4624](/Assets/Scenario_2/4624.png)

---

## 🟦 9. Post-Compromise Activity — User-Level Reconnaissance

After gaining access, the attacker performed standard user-level reconnaissance.

Observed commands included:
- whoami
- hostname
- systeminfo
- ipconfig /all
- local environment inspection

These actions are typical of early-stage attacker reconnaissance.

[SCREENSHOT PLACEHOLDER]
Windows PowerShell — Initial reconnaissance commands
![xfreerdp3](/Assets/Scenario_2/xfreerd3_powershell.png)

---

## 🟥 10. Privileged Action Attempts — Blocked but Ignored

The attacker attempted several actions requiring elevated privileges, including:
- Registry access to SAM and SYSTEM hives
- Local account enumeration
- Network and domain queries

All privileged actions failed due to insufficient permissions.
The attacker confirmed they only had standard user access.

Evidence of failed privileged registry access is visible at the top of the reconnaissance screenshot referenced in Section 9.

No SOC investigation occurred.

---

## 🧠 11. Attacker Decision Point

At this stage, the attacker assessed that:
- Privilege escalation was not immediately possible
- Continued activity would increase detection risk
- The compromised credentials remained valid

The attacker ceased active operations and disconnected, preserving access for potential future use.

This behavior reflects realistic attacker tradecraft.

---

## 📉 12. SIEM Summary — What Was Seen

Observed:
- OSINT-based credential guessing
- High-volume NTLM authentication failures
- Successful RDP authentication
- User-level reconnaissance activity
- Failed privileged action attempts

Missing:
- Alert generation
- SOC triage or investigation
- Credential reset
- Containment or remediation

---

## 🟧 13. Scenario 2 Outcome

This scenario demonstrates that:
- Visibility alone does not equal security
- Logs without response are ineffective
- Attackers can remain undetected even when activity is recorded
- Failure to act enables future compromise

The compromised account remains active and unchanged.

---

## 🔐 14. Post-Incident Hardening (Implemented After Scenario 2)

Following this incident, the organization implemented delayed security improvements:

- Account lockout policy enabled (5 attempts / 30 minutes)
- Custom Wazuh alert rules created for:
  - Multiple failed logons
  - Successful logon after failed attempts
  - NTLM authentication usage
- Correlation rules introduced to link failed and successful authentication events
- Improved visibility into process creation (Event ID 4688)
- SOC dashboards created to highlight high-risk authentication behavior

These changes directly enable Scenario 3, where the SOC finally responds in real time.

---

## 🔜 Lead-In to Scenario 3

Because the attacker was not contained:
- Valid credentials remain usable
- No proactive defense exists
- The SOC is forced into a reactive posture

This leads directly into Scenario 3: Active Response and Containment.

---

## ✔ End of Scenario 2

Snapshots:
- Windows → scenario2_end_windows
- Kali → scenario2_end_kali
