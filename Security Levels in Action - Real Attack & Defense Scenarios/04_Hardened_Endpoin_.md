# 🔒 Hardened Endpoint — Post-Incident Technical Hardening

This section describes the **concrete technical hardening steps** applied after Scenario 3.  
No theory, no best practices fluff — only **what was changed, why, and what it prevents**.

The goal of hardening was **not to make the system “unhackable”**, but to:
- reduce attack surface
- increase detection depth
- shorten attacker dwell time
- prevent repeat compromise using the same techniques

---

## 1️⃣ Account & Authentication Hardening

### 🔐 Password Policy (Already Partially Enforced)
Confirmed and kept in place:

- Minimum password length: **10**
- Password history: **5**
- Maximum password age: **90 days**
- Complexity requirements: **Enabled**

**Security impact:**
- Prevents reuse of OSINT-derived or previously leaked credentials
- Increases cost of targeted password guessing

---

### 🔒 Account Lockout Policy (Now Enforced)

**Final configuration:**
- Account lockout threshold: **5 failed attempts**
- Reset account lockout counter after: **30 minutes**
- Account lockout duration: **30 minutes**

**Security impact:**
- Prevents extended brute-force and password spraying
- Forces attacker to either:
  - rotate accounts
  - slow down significantly
  - abandon the attack

This directly addresses the weakness exploited in Scenario 2.

---

## 2️⃣ Remote Access Hardening (RDP)

### 🚫 RDP Exposure Reduction

**Changes applied:**
- IP-based access restrictions enforced
- Attacker source IPs blocked at endpoint level
- No longer fully open to the internet

**Security impact:**
- Eliminates opportunistic scanning and brute-force attempts
- Forces attacker to control infrastructure within allowed ranges

---

### 🔑 Credential Hygiene

**Action taken:**
- Compromised user password reset immediately
- Active sessions forcibly terminated

**Security impact:**
- Invalidates stolen credentials instantly
- Prevents session hijacking or reuse

---

## 3️⃣ Logging & Telemetry Improvements

### 📊 Windows Security Logging

**Confirmed enabled:**
- Successful logons (4624)
- Failed logons (4625)
- NTLM authentication events
- RDP authentication visibility

**Security impact:**
- Authentication misuse is now visible in real time
- Enables correlation and severity escalation in SIEM

---

### 🧠 PowerShell Visibility Upgrade

**Change applied:**
- PowerShell **Script Block Logging enabled**
- PowerShell Operational log collected by Wazuh

**Resulting telemetry:**
- Event ID **4104**
- Full PowerShell command content
- Script execution context

**Security impact:**
- Eliminates “PowerShell blind spot”
- Allows SOC to:
  - reconstruct attacker intent
  - distinguish benign admin activity from malicious recon
  - detect living-off-the-land techniques

---

## 4️⃣ Sysmon Telemetry Alignment

### ⚙ Sysmon Configuration Review

**Maintained telemetry:**
- Process creation
- File creation
- Network connections
- Registry modifications

**Adjustment made:**
- Noise reduction for non-security-relevant paths
- Focus on user-writable directories and PowerShell execution

**Security impact:**
- Higher signal-to-noise ratio
- Faster analyst triage
- Reduced alert fatigue

---

## 5️⃣ SIEM & Detection Logic Improvements

### 🚨 Alert Threshold Tuning

**Changes:**
- Authentication alerts aligned with lockout policy
- Correlation rules detect:
  - failed → successful logon patterns
  - IP change attempts
  - post-containment retry behavior

**Security impact:**
- Prevents false positives
- Escalates alerts only when behavior is meaningful
- Reflects real attacker tradecraft

---

### 📈 Severity Normalization

**Outcome:**
- Low-risk noise downgraded
- High-confidence events elevated
- Dashboard reflects **actual risk**, not raw volume

---

## 6️⃣ Operational SOC Improvements

### 🧑‍💻 Response Workflow Defined

**SOC actions now standardized:**
1. Session termination
2. Credential reset
3. IP containment
4. Post-containment monitoring

**Security impact:**
- Faster response time
- Fewer decision points during live incidents
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

After hardening:
- Reuse of compromised credentials **fails**
- IP-based evasion **fails**
- Recon activity is **visible**
- Attacker dwell time is **minimal**
- Repeat compromise using the same path is **not viable**

This hardened state represents a **realistic, production-ready endpoint security posture**, not an artificial lab configuration.

---

# ✔ End of Hardened Endpoint Section
