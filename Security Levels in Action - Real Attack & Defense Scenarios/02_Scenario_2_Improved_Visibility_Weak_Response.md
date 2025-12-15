# 🟧 Scenario 2 — Good SIEM, Weak SOC Response  
**Attacker gains access again — visibility improves, but no action is taken**

---

## 🧭 Scenario Context

This scenario continues directly after Scenario 1.

Following the initial compromise, the organization implemented **partial security improvements**:
- Logging and telemetry were significantly enhanced
- A SIEM is now actively ingesting Windows security events
- Sysmon was deployed
- Basic hardening steps were taken

However, **no SOC processes, alerting logic, or response workflows were introduced**.

Scenario 2 demonstrates a common real-world failure:
> **The attacker is clearly visible — but ignored.**

---

## 🟦 1. Environment Overview — After Scenario 1 Improvements

The Windows Server endpoint remains publicly accessible and operational as a workstation-style endpoint.

### 🔐 Password & Account Security (Partially Improved)

- Password complexity: **Enabled**
- Minimum password length: **10**
- Password history: **5**
- Maximum password age: **90 days**
- Account lockout policy: **Disabled**
- MFA: **Not implemented**

A real employee user changed their password to a **personal, OSINT-derived password**, making it stronger than default credentials — but still guessable using targeted reconnaissance.

---

## 📊 Logging & Visibility — Improved but Incomplete

### ✔ Advanced Audit Policy Configuration

The following audit categories were enabled:

- **Logon / Logoff**
  - Successful logons
  - Failed logons
- **Account Logon**
  - Credential validation
- **Account Management**
  - User and group changes
- **Detailed Tracking**
  - Process creation

These changes significantly increased authentication and activity visibility.

---

## 👁️ Sysmon Deployment

Sysmon was installed using a community-recommended baseline configuration.

Active telemetry includes:
- Process creation
- Network connections
- File creation
- Registry modifications

Sysmon events are forwarded to the SIEM via the Wazuh agent.

---

## 📡 Wazuh Agent & SIEM State

- Windows Security Event logs are forwarded
- NTLM authentication failures are visible
- RDP logons are visible
- Limited process execution telemetry is available

⚠️ No alert rules, thresholds, or SOC playbooks are configured.

---

## 🧱 Network Exposure (Unchanged)

- RDP (3389) is publicly exposed
- No IP allowlisting
- No rate limiting
- No brute-force detection rules
- No account lockout

---

# 🟧 2. Initial Access — OSINT-Based Credential Attack

### Attacker Preparation

Using data gathered during the OSINT Pre-Stage, the attacker prepared:

- A custom username list based on corporate naming conventions
- A targeted password list derived from personal information

Files used:
- `usernames.txt`
- `osint_passwords.txt`

---

## 🔴 Authentication Attack Observed in SIEM

The attacker initiated repeated authentication attempts over RDP.

### SIEM Evidence

- **Event ID 4776** — NTLM credential validation failures
- Status code: **0xC000006A** (Wrong password)
- Multiple target usernames tested
- High-volume authentication noise observed

This confirms:
- Valid usernames were discovered
- Password guessing was actively occurring
- No defensive controls interrupted the attack

---

## 🟢 Successful Authentication

After repeated failures, the attacker successfully authenticated using OSINT-derived credentials.

### Evidence

- Successful RDP logon observed
- Multiple successful logon events due to RDP reconnect behavior
- Same source IP as failed attempts

⚠️ No alert was generated.  
⚠️ No SOC investigation followed.

---

# 🟦 3. Post-Compromise Activity — User-Level Access

After obtaining access, the attacker performed standard reconnaissance actions.

### Commands Executed

- Identity and system checks:
  - `whoami`
  - `hostname`
  - `systeminfo`

- Environment enumeration:
  - Directory listing of user profiles
  - Running processes and services
  - Network configuration (`ipconfig /all`)

These actions are typical of early-stage attacker reconnaissance.

---

## 📉 Visibility Gap Identified

While authentication activity was clearly visible, **detailed post-compromise command execution telemetry was limited**.

As a result:
- The SIEM confirms access occurred
- The SIEM cannot fully reconstruct attacker intent
- No alerting logic escalated the activity

This represents a **critical detection maturity gap**.

---

# 🟥 4. Privileged Action Attempts — Blocked but Ignored

The attacker attempted several actions requiring administrative privileges:

- Registry access to SAM and SYSTEM hives
- Local account manipulation
- Scheduled task creation
- Event log clearing

### Result

- All privileged actions failed due to insufficient permissions
- Access was denied at the operating system level
- The attacker confirmed they only had standard user access

⚠️ These failed attempts were **not investigated by the SOC**.

---

# 🧠 5. Attacker Decision Point

At this stage, the attacker assessed that:

- Privilege escalation was not immediately possible
- Continued activity would increase detection risk
- The compromised credentials remained valid

### Attacker Action

The attacker **ceased active operations** and disconnected, preserving access for potential future use.

This behavior reflects realistic attacker tradecraft.

---

# 🟦 6. SIEM Summary — What Was Seen

✔ OSINT-based credential guessing  
✔ High-volume NTLM authentication failures  
✔ Successful RDP logon  
✔ User-level reconnaissance activity  
✔ Failed privileged action attempts  

❌ No alert generation  
❌ No SOC response  
❌ No account reset  
❌ No containment  

---

# 🟧 7. Scenario 2 Outcome

This scenario demonstrates that:

- Visibility alone does not equal security
- Logs without response are ineffective
- Attackers can remain undetected even when activity is recorded
- Failure to act enables future compromise

The compromised account remains active and unchanged.

---

# 🔜 Lead-In to Scenario 3

Because the incident was not properly handled:

- The attacker retains valid credentials
- No lessons were operationalized
- Security posture remains reactive

This directly enables **Scenario 3**, where the attacker returns and the SOC is finally forced to respond.

---

# ✔ End of Scenario 2

**Snapshots:**
- Windows → `scenario2_end_windows`
- Kali → `scenario2_end_kali`
