# 🟥 Scenario 1 — Weak Endpoint  
**I gained access due to weak credentials and ineffective security visibility**

---

## 🟦 Environment Overview — Initial State

Before I began the attack, the Windows Server endpoint was intentionally left in a weak, low-maturity security state.

Although the operating system is **Windows Server 2022**, I treated it purely as an **endpoint / workstation**, not as a hardened enterprise server.

This setup represents a **small logistics company** with limited security maturity, no SOC team, and no active detection or response processes.

---

## 🔐 Password & Account Security — Initial State

The endpoint used a **severely misconfigured password and account policy**.

Current configuration:
- Password complexity: Enabled
- Minimum password length: 8
- Maximum password age: Unlimited
- Account lockout threshold: Disabled
- Lockout duration: 0
- Lockout observation window: 0

Impact:
- Weak and predictable passwords were allowed
- Unlimited authentication attempts were possible
- Password spraying and brute-force attacks were feasible without interruption
- No user-side or system-side deterrence existed

---

## 🌐 Network Exposure

The Windows endpoint was **directly exposed to the internet**.

Externally observed services:

| Port | Service | State | Notes |
|----|----|----|----|
| 3389/tcp | RDP | Open | Publicly accessible |
| 5986/tcp | WinRM HTTPS | Open | Exposed externally |
| 445/tcp | SMB | Filtered | Likely blocked by firewall |
| 5985/tcp | WinRM HTTP | Filtered | Not directly reachable |
| 22/tcp | SSH | Closed | Not exposed |

Risk summary:
- RDP was publicly reachable
- No MFA was enforced
- No IP allowlisting was configured
- No rate limiting was in place
- No brute-force protection existed

![Nmap scan](/Assets/Scenario_1/Nmap.png)

---

## 📊 Logging & Visibility — Initial State

Windows auditing was **minimally configured**.

Audit coverage:
- Logon auditing: Partially enabled
- Process creation auditing: Disabled
- Object access auditing: Disabled
- Policy change auditing: Disabled
- File system auditing: Disabled

Resulting visibility:
- Authentication events lacked context
- Failed and successful logons blended together
- My activity was indistinguishable from background noise
- No reliable detection capability existed

---

## 👁️ Sysmon — Initial State

Sysmon was **not installed**.

Without Sysmon:
- No process creation telemetry existed
- No network connection visibility was available
- No file creation or modification events were logged
- No registry activity monitoring was present

Large portions of my activity were completely invisible.

---

## 📡 Wazuh Agent — Initial State

The Wazuh agent was installed but **poorly configured**.

Observed state:
- Limited Windows Security Event forwarding
- No Sysmon integration
- No File Integrity Monitoring
- No meaningful correlation rules
- No alerting logic

From a SOC perspective:
- Logs technically existed
- No actionable signal was produced
- No alerts were generated

---

# 🟥 Attack Execution

---

## 1️⃣ Step 1 — Reconnaissance

I performed basic network reconnaissance to identify exposed services.

Actions:
- Port scanning from my attacker system
- Focus on common remote access services

Observed result:
- RDP was exposed and reachable from the internet

![Nmap scan](/Assets/Scenario_1/Nmap.png)

---

## 2️⃣ Step 2 — Username Enumeration

I prepared a basic username list using common enterprise naming patterns.

Usernames tested:  
administrator  
admin  
user  
employee  
employee1  

This reflects a realistic early-stage attacker assumption phase.

![Users list](/Assets/Scenario_1/Users.png)

---

## 3️⃣ Step 3 — Password Spraying

I created a simple password list using extremely common credentials.

Passwords tested:  
Welcome1  
Admin123  
Qwerty123  
Password123  
Password1  

![Password list](/Assets/Scenario_1/Passwords.png)

Attack method:
- Password spraying via RDP
- No lockout policy was triggered
- No defensive response was observed

Result:
- Valid credentials were discovered

Discovered credentials:
- Username: employee1
- Password: Password123

![Password spraying](/Assets/Scenario_1/Password_Spraying.png)

---

## 4️⃣ Step 4 — RDP Login

I authenticated successfully using the discovered credentials.

Result:
- User-level access was obtained
- No security warnings appeared
- No account lockout occurred
- No alerting was triggered

---

## 5️⃣ Step 5 — Post-Compromise Reconnaissance

After gaining access, I performed basic reconnaissance.

Activities:
- Identity confirmation
- Hostname identification
- OS and system information gathering
- Local user enumeration

Outcome:
- System ownership was confirmed
- The local environment was mapped
- No detection was triggered

![RDP login and reconnaissance](/Assets/Scenario_1/xfreerdp3-login+reconnaisence.png)

---

## 6️⃣ Step 6 — Simulated Ransomware Impact

To demonstrate business impact **without deploying real malware**, I performed a simulated ransomware action.

Actions:
- Files were renamed to simulate encryption
- A fake ransom note was placed on the desktop
- Cryptocurrency payment was demanded

Important notes:
- No real malware was used
- No encryption occurred
- This step exists solely to demonstrate **impact and disruption**

![Fake ransomware and left behind message](/Assets/Scenario_1/Ransomware+left_behind_message.png)

---

## 7️⃣ Step 7 — Attacker Exit

I exited the environment cleanly.

Actions:
- Normal logout
- No cleanup performed
- No traces intentionally removed

---

## 🟨 SIEM Visibility During the Attack

My initial SIEM review suggested **no meaningful visibility**.

After correcting the time window to match the attack period:
- Approximately 1 hour of activity was visible
- 1,387 events were ingested
- Most events were authentication noise
- No alerts were triggered
- No prioritization or correlation occurred

Key takeaway:
High log volume without filtering caused **malicious activity to disappear in noise**.

![Noisy Wazuh dashboard](/Assets/Scenario_1/Noisy_wazuh.png)

---

## 🟩 Post-Incident Improvements

After completing the attack, I applied **partial security hardening**.

---

### 🔐 Password Policy Improvements

Changes applied:
- Minimum password length increased to 10
- Password history enforced to 5
- Maximum password age configured to 90 days

Unchanged:
- Account lockout policy remained disabled

This represents an **incomplete but realistic** improvement step.

![Hardened password policy](/Assets/Scenario_1/Hardened_password_policy.png)

---

### 👁️ Sysmon Deployment

I installed Sysmon using the **SwiftOnSecurity** configuration.

Purpose:
- Improve process visibility
- Capture network activity
- Log file and registry changes

Verification:
- Sysmon service running successfully

![Sysmon running](/Assets/Scenario_1/Sysmon_running.png)

---

### 📊 Audit Policy & Authentication Visibility Improvements

After the initial compromise, I identified a critical lack of authentication and activity visibility.  
Although logs technically existed, advanced audit telemetry was not enabled, preventing meaningful detection and correlation.

To address this, I enabled and hardened **Advanced Audit Policy Configuration**.

---

#### 🔐 Advanced Audit Policy Enforcement

I enabled the following policy to ensure granular audit settings take precedence over legacy categories:

- **Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings**
  - Status: **Enabled**

This ensured that all configured Advanced Audit subcategories were actively enforced.

---

#### 🔑 Authentication & Logon Auditing

I enabled the following audit categories to improve visibility into authentication activity:

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

To detect unauthorized account creation and privilege escalation, I enabled:

- **Account Management → Audit User Account Management**
  - Success  
  *(Generates Event IDs such as 4720 and 4732)*

This enables tracking of local account changes that may indicate persistence or privilege abuse.

---

#### ⚙️ Process & Command Execution Visibility

To gain insight into post-compromise activity, I enabled:

- **Detailed Tracking → Audit Process Creation**
  - Success  
  *(Generates Event ID 4688)*

This allows reconnaissance commands and tooling execution to be visible and correlated with user sessions.

---

#### 🗓️ Scheduled Task & Object Activity Auditing

To improve visibility into potential persistence mechanisms, I partially enabled:

- **Object Access → Audit Other Object Access Events**
  - Success  
  *(Enables detection of scheduled task creation, Event ID 4698)*

---

#### 📡 SIEM Integration Validation

After applying these changes, I confirmed that failed and successful logon events were:
- Visible in Windows Security Event Logs
- Successfully forwarded by the Wazuh agent
- Ingested and searchable in the Wazuh SIEM

This confirmed that authentication and account activity telemetry was now available.

---

#### ⚠️ Security Maturity Gap

Although logging and visibility were significantly improved, **I did not introduce alerting rules or SOC response processes at this stage**.

As a result:
- Malicious activity became visible
- SIEM telemetry was rich and reliable
- **But my activity could still proceed without interruption**

This gap directly leads into **Scenario 2**, where I successfully compromise the system again despite improved logging, due to the absence of active SOC response.

---

## 🧠 Scenario 1 — Final Outcome

This scenario demonstrates:
- How a single exposed service can compromise an endpoint
- How weak credentials negate perimeter security
- How logging without structure creates blind spots
- Why visibility alone is not enough without detection logic

Scenario 1 establishes the baseline failure state  
and motivates the security improvements that follow.
