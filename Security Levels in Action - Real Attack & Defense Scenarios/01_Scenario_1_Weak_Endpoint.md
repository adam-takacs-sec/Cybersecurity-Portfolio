# 🟥 Scenario 1 — Weak Endpoint
**Attacker gains access due to weak credentials and ineffective security visibility**

---

## 🟦 Environment Overview — Initial State

Before the attack begins, the Windows Server endpoint is intentionally left in a weak, low-maturity security state.

Although the operating system is **Windows Server 2022**, it is treated purely as an **endpoint / workstation**, not as a hardened enterprise server.

This setup represents a **small logistics company** with limited security maturity, no SOC team, and no active detection or response processes.

---

## 🔐 Password & Account Security — Initial State

The endpoint uses a **severely misconfigured password and account policy**.

Current configuration:
- Password complexity: Enabled
- Minimum password length: 8
- Maximum password age: Unlimited
- Account lockout threshold: Disabled
- Lockout duration: 0
- Lockout observation window: 0

Impact:
- Weak and predictable passwords are allowed
- Unlimited authentication attempts are possible
- Password spraying and brute-force attacks are feasible without interruption
- No user-side or system-side deterrence exists

---

## 🌐 Network Exposure

The Windows endpoint is **directly exposed to the internet**.

Externally observed services:

| Port | Service | State | Notes |
|----|----|----|----|
| 3389/tcp | RDP | Open | Publicly accessible |
| 5986/tcp | WinRM HTTPS | Open | Exposed externally |
| 445/tcp | SMB | Filtered | Likely blocked by firewall |
| 5985/tcp | WinRM HTTP | Filtered | Not directly reachable |
| 22/tcp | SSH | Closed | Not exposed |

Risk summary:
- RDP is publicly reachable
- No MFA
- No IP allowlisting
- No rate limiting
- No brute-force protection

![Nmap scan](/Assets/Scenario_1/Nmap.png)

---

## 📊 Logging & Visibility — Initial State

Windows auditing is **minimally configured**.

Audit coverage:
- Logon auditing: Partially enabled
- Process creation auditing: Disabled
- Object access auditing: Disabled
- Policy change auditing: Disabled
- File system auditing: Disabled

Resulting visibility:
- Authentication events lack context
- Failed and successful logons blend together
- Attacker activity is indistinguishable from background noise
- No reliable detection capability exists

---

## 👁️ Sysmon — Initial State

Sysmon is **not installed**.

Without Sysmon:
- No process creation telemetry
- No network connection visibility
- No file creation or modification events
- No registry activity monitoring

Large portions of attacker behavior are completely invisible.

---

## 📡 Wazuh Agent — Initial State

The Wazuh agent is installed but **poorly configured**.

Observed state:
- Limited Windows Security Event forwarding
- No Sysmon integration
- No File Integrity Monitoring
- No meaningful correlation rules
- No alerting logic

From a SOC perspective:
- Logs technically exist
- No actionable signal is produced
- No alerts are generated

---

# 🟥 Attack Execution

---

## 1️⃣ Step 1 — Reconnaissance

The attacker performs basic network reconnaissance to identify exposed services.

Action:
- Port scanning from the attacker system
- Focus on common remote access services

Observed result:
- RDP is exposed and reachable from the internet

![Nmap scan](/Assets/Scenario_1/Nmap.png)

---

## 2️⃣ Step 2 — Username Enumeration

A basic username list is prepared using common enterprise naming patterns.

Usernames tested:  
administrator  
admin  
user  
employee  
employee1  

This reflects a realistic attacker assumption phase.

![Users list](/Assets/Scenario_1/Users.png)

---

## 3️⃣ Step 3 — Password Spraying

A simple password list is created using extremely common credentials.

Passwords tested:  
Welcome1  
Admin123  
Qwerty123  
Password123  
Password1  

![Password list](/Assets/Scenario_1/Passwords.png)

Attack method:
- Password spraying via RDP
- No lockout policy triggered
- No defensive response observed

Result:
- Valid credentials discovered

Discovered credentials:
- Username: employee1
- Password: Password123

![Password spraying](/Assets/Scenario_1/Password_Spraying.png)

---

## 4️⃣ Step 4 — RDP Login

The attacker authenticates successfully using the discovered credentials.

Result:
- User-level access obtained
- No security warnings
- No account lockout
- No alerting

---

## 5️⃣ Step 5 — Post-Compromise Reconnaissance

After access is obtained, the attacker performs basic reconnaissance.

Activities:
- Identity confirmation
- Hostname identification
- OS and system information gathering
- Local user enumeration

Outcome:
- System ownership confirmed
- Local environment mapped
- No detection triggered

![RDP login and reconnaissance](/Assets/Scenario_1/xfreerdp3-login+reconnaisence.png)

---

## 6️⃣ Step 6 — Simulated Ransomware Impact

To demonstrate business impact **without deploying real malware**, a simulated ransomware action is performed.

Action:
- Files are renamed to simulate encryption
- A fake ransom note is placed on the desktop
- Cryptocurrency payment is demanded

Important:
- No real malware is used
- No encryption occurs
- This step exists solely to demonstrate **impact and disruption**

![Fake ransomware and left behind message](/Assets/Scenario_1/Ransomware+left_behind_message.png)

---

## 7️⃣ Step 7 — Attacker Exit

The attacker exits the environment cleanly.

Action:
- Normal logout
- No cleanup required
- No traces intentionally removed

---

## 🟨 SIEM Visibility During the Attack

Initial SIEM review suggested **no meaningful visibility**.

After correcting the time window to match the attack period:
- Approximately 1 hour of activity
- 1,387 events ingested
- Mostly authentication noise
- No alerts triggered
- No prioritization
- No correlation

Key takeaway:
High log volume without filtering causes **malicious activity to disappear in noise**.

![Noisy Wazuh dashboard](/Assets/Scenario_1/Noisy_wazuh.png)

---

## 🟩 Post-Incident Improvements

After the incident, **partial security hardening** is applied.

---

### 🔐 Password Policy Improvements

Changes applied:
- Minimum password length increased to 10
- Password history enforced to 5
- Maximum password age configured to 90 days

Unchanged:
- Account lockout policy remains disabled

This represents an **incomplete but realistic** improvement step.

![Hardened password policy](/Assets/Scenario_1/Hardened_password_policy.png)

---

### 👁️ Sysmon Deployment

Sysmon is installed using the **SwiftOnSecurity** configuration.

Purpose:
- Improve process visibility
- Capture network activity
- Log file and registry changes

Verification:
- Sysmon service running successfully

![Sysmon running](/Assets/Scenario_1/Sysmon_running.png)

---

### 📊 Audit Policy & Authentication Visibility Improvements

After the initial compromise the organization identified a critical lack of authentication and activity visibility.
Although logs technically existed, advanced audit telemetry was not enabled, preventing meaningful detection and correlation.

To address this, **Advanced Audit Policy Configuration** was enabled and selectively hardened.

---

#### 🔐 Advanced Audit Policy Enforcement

To ensure granular audit settings take precedence over legacy audit categories, the following policy was enabled:

- **Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings**
  - Status: **Enabled**

This guarantees that all configured Advanced Audit subcategories are actively enforced by the system.

---

#### 🔑 Authentication & Logon Auditing

The following audit categories were enabled to improve visibility into authentication activity, including brute-force attempts and successful remote access:

- **Logon/Logoff → Audit Logon**
  - Success
  - Failure  
  *(Generates Event IDs 4624 and 4625)*

- **Logon/Logoff → Audit Logoff**
  - Success

- **Account Logon → Audit Credential Validation**
  - Success
  - Failure  
  *(Provides additional context for authentication failures)*

These changes allow failed and successful RDP logons to be clearly distinguished and correlated in the SIEM.

---

#### 👤 Account & Privilege Management Auditing

To detect unauthorized account creation and privilege escalation, account management auditing was enabled:

- **Account Management → Audit User Account Management**
  - Success  
  *(Generates Event IDs such as 4720 – user creation, and 4732 – user added to Administrators group)*

This enables tracking of local account changes that may indicate persistence or privilege abuse.

---

#### ⚙️ Process & Command Execution Visibility

To gain insight into post-compromise attacker activity, detailed process tracking was enabled:

- **Detailed Tracking → Audit Process Creation**
  - Success  
  *(Generates Event ID 4688)*

This allows basic reconnaissance commands and tooling execution to be visible and correlated with user sessions.

---

#### 🗓️ Scheduled Task & Object Activity Auditing

To improve visibility into potential persistence mechanisms, object access auditing was partially enabled:

- **Object Access → Audit Other Object Access Events**
  - Success  
  *(Enables detection of scheduled task creation, Event ID 4698)*

---

#### 📡 SIEM Integration Validation

After applying these changes, failed and successful logon events were confirmed to be:

- Visible in **Windows Security Event Logs**
- Successfully forwarded by the **Wazuh agent**
- Ingested and searchable in the **Wazuh SIEM**

This confirms that authentication and account activity telemetry is now available for detection and investigation.

---

#### ⚠️ Security Maturity Gap

Although logging and visibility were significantly improved, **no alerting rules or SOC response processes were introduced at this stage**.

As a result:
- Malicious activity is now visible
- SIEM telemetry is rich and reliable
- **But attacker behavior can still proceed without interruption**

This gap directly leads into **Scenario 2**, where the attacker successfully compromises the system again despite improved logging, due to the absence of active SOC response.

---

## 🧠 Scenario 1 — Final Outcome

This scenario demonstrates:

- How a single exposed service can compromise an endpoint
- How weak credentials negate perimeter security
- How logging without structure creates blind spots
- Why visibility alone is not enough without detection logic

Scenario 1 establishes the baseline failure state  
and motivates the security improvements.
