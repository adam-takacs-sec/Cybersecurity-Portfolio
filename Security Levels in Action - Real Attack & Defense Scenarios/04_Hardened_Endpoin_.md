# 🔒 Hardened Endpoint — Post-Incident Technical Hardening

In this phase, I hardened the endpoint **after Scenario 3**, based directly on the attack paths, detection gaps, and operational weaknesses I observed.

This is **not a best-practice checklist** and **not theoretical guidance**.  
Every change listed below was implemented to **close a real gap that was exploited or observed during the scenarios**.

The goal of this hardening phase was:
- to prevent repeat compromise using the same techniques
- to reduce attacker dwell time
- to increase detection depth
- to make SOC response faster and more reliable

---

## 1️⃣ Account & Authentication Hardening

### 🔐 Password Policy (Validated and Kept)

I confirmed and kept the improved password policy introduced after Scenario 1:

- Minimum password length: **10**
- Password history: **5**
- Maximum password age: **90 days**
- Password complexity: **Enabled**

**Why this matters:**
- Prevents reuse of OSINT-derived passwords
- Increases cost of targeted credential guessing
- Forces periodic credential rotation

This directly addresses the OSINT-based credential compromise seen in Scenario 2.

---

### 🔒 Account Lockout Policy (Enforced)

I enabled and aligned the account lockout policy with realistic enterprise settings:

- Account lockout threshold: **5 failed attempts**
- Lockout duration: **30 minutes**
- Reset counter after: **30 minutes**

**Security impact:**
- Prevents extended password spraying and brute-force attacks
- Forces attackers to either:
  - slow down significantly
  - rotate infrastructure
  - abandon the attack

This directly blocks the attack pattern used in Scenario 2.

---

## 2️⃣ Remote Access Hardening (RDP)

### 🚫 RDP Exposure Reduction

I reduced unnecessary exposure of the RDP service:

- Applied IP-based restrictions to RDP
- Blocked attacker source IPs at the endpoint level
- Eliminated unrestricted public RDP access

**Security impact:**
- Stops opportunistic scanning and automated attacks
- Forces attackers to operate from constrained infrastructure
- Makes brute-force and spray attacks significantly harder

---

### 🔑 Credential Hygiene & Session Control

After detecting compromise in Scenario 3, I performed:

- Immediate password reset for the compromised account
- Forced termination of all active RDP sessions

**Security impact:**
- Instantly invalidates stolen credentials
- Prevents session reuse or hijacking
- Limits attacker dwell time to minutes instead of hours

---

## 3️⃣ Logging & Telemetry Hardening

### 📊 Windows Security Logging

I validated that the following security events were fully enabled and collected:

- Successful logons (**4624**)
- Failed logons (**4625**)
- NTLM authentication activity
- RDP logon events

**Security impact:**
- Authentication misuse is visible in real time
- Enables meaningful correlation in the SIEM
- Allows distinction between noise and attack behavior

---

### 🧠 PowerShell Visibility Upgrade

To eliminate the PowerShell blind spot observed in Scenario 3, I enabled:

- **PowerShell Script Block Logging**
- Collection of PowerShell Operational logs by Wazuh

This introduced:
- Event ID **4104**
- Full PowerShell command content
- Script execution context

**Security impact:**
- Enables reconstruction of attacker intent
- Allows detection of living-off-the-land techniques
- Distinguishes benign admin activity from malicious reconnaissance

---

## 4️⃣ Sysmon Telemetry Alignment

### ⚙ Sysmon Configuration Review

I reviewed and tuned the Sysmon configuration to balance visibility and noise.

**Telemetry maintained:**
- Process creation
- Network connections
- File creation
- Registry modifications

**Adjustments made:**
- Reduced noise from known benign system paths
- Focused monitoring on:
  - user-writable directories
  - PowerShell execution
  - suspicious parent-child process relationships

**Security impact:**
- Higher signal-to-noise ratio
- Faster SOC triage
- Reduced alert fatigue

---

## 5️⃣ SIEM & Detection Logic Improvements

### 🚨 Alert Threshold Alignment

I aligned detection logic with the enforced security policies:

- Authentication alert thresholds matched lockout policy
- Correlation rules detect:
  - failed → successful logon patterns
  - authentication attempts after IP blocking
  - attacker retry behavior after containment

**Security impact:**
- Prevents false positives
- Escalates alerts only when behavior is meaningful
- Reflects real attacker tradecraft instead of raw event volume

---

### 📈 Severity Normalization

I normalized alert severity across the SIEM:

- Low-risk authentication noise downgraded
- High-confidence attack patterns elevated
- Dashboards now reflect **actual risk**, not event count

This ensures that SOC attention is directed where it matters.

---

## 6️⃣ SOC Operational Hardening

### 🧑‍💻 Response Workflow Definition

Based on Scenario 3, I formalized a clear SOC response workflow:

1. Identify suspicious authentication
2. Terminate active sessions
3. Reset affected credentials
4. Apply IP-based containment
5. Monitor for post-containment retries

**Security impact:**
- Faster response during live incidents
- Reduced decision-making overhead
- Consistent handling of repeated attacks

---

## 7️⃣ Attack Surface Reduction Summary

| Area | Before | After |
|----|----|----|
| RDP Exposure | Public | Restricted |
| Lockout Policy | Disabled | Enforced |
| Credential Reuse | Possible | Prevented |
| PowerShell Visibility | Process-only | Command-level |
| Detection Logic | Passive | Correlated |
| SOC Response | Ad-hoc | Defined |

---

## 🟢 Hardened Endpoint State

After completing the hardening phase:

- Reuse of compromised credentials **fails**
- IP-based evasion **fails**
- Reconnaissance activity is **visible**
- Attacker dwell time is **minimal**
- Repeat compromise using the same attack path is **not viable**

This hardened state represents a **realistic, production-ready endpoint security posture**, not an artificial lab configuration.

---

# ✔ End of Hardened Endpoint Section
