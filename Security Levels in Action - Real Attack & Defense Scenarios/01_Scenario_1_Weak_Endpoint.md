# Scenario 1 — Weak Endpoint
Attacker gains access due to weak credentials and ineffective security visibility

## Environment Overview — Initial State

Before the attack begins, the Windows Server endpoint is intentionally left in a weak, low-maturity security state.

Although the operating system is Windows Server 2022, it is treated purely as an endpoint / workstation and not as a hardened enterprise server.

This setup represents a small logistics company with limited security maturity and no active SOC processes.

---

## Password & Account Security — Initial State

The endpoint uses a severely misconfigured password and account policy.

- Password complexity: Disabled
- Minimum password length: 0
- Maximum password age: Unlimited
- Account lockout threshold: Disabled
- Lockout duration: 0
- Lockout observation window: 0

Impact:
- Very weak passwords are allowed
- Unlimited authentication attempts are possible
- Password spraying and brute-force attacks are feasible without interruption

Note:
The hardened password policy screenshot belongs to the post-incident improvement section.
The weak baseline is already documented during the OSINT pre-stage.

---

## Network Exposure

The Windows endpoint is directly exposed to the internet.

Externally observed services:
- TCP 3389 — RDP — Open
- TCP 5986 — WinRM HTTPS — Open
- TCP 445 — SMB — Filtered
- TCP 5985 — WinRM HTTP — Filtered
- TCP 22 — SSH — Closed

Risk:
RDP is publicly accessible without MFA, IP allowlisting, or rate limiting.

Screenshot:
Nmap.png

---

## Logging & Visibility — Initial State

Windows auditing is minimally configured.

- Logon auditing: partially enabled
- Process creation auditing: disabled
- Object access auditing: disabled
- Policy change auditing: disabled
- File system auditing: disabled

Result:
- Authentication events lack context
- Attacker activity blends into background noise
- No meaningful detection capability exists

---

## Sysmon — Initial State

Sysmon is not installed.

Without Sysmon:
- No process creation telemetry
- No network connection visibility
- No file creation or modification events
- No registry activity monitoring

Large portions of attacker activity are invisible.

---

## Wazuh Agent — Initial State

The Wazuh agent is installed but poorly configured.

- Limited Windows Security Event forwarding
- No Sysmon integration
- No File Integrity Monitoring
- No meaningful correlation rules
- No alerting

From a SOC perspective, logs exist but provide no actionable signal.

---

## Attack Execution

### Step 1 — Reconnaissance

Port scanning from the attacker system confirms RDP exposure.

Command executed:
nmap -Pn -p 3389,445,5985,5986 <WINDOWS_IP>

Screenshot:
Nmap.png

---

### Step 2 — Username Enumeration

A basic username list is prepared based on common enterprise patterns.

Users tested:
administrator
admin
user
employee
employee1

Screenshot:
Users.png

---

### Step 3 — Password Spraying

A simple password list is created using common weak credentials.

Passwords tested:
Welcome1
Admin123
Qwerty123
Password123
Password1

Screenshot:
Passwords.png

Password spraying attack executed:
hydra -L users.txt -P passwords.txt rdp://<WINDOWS_IP>

Result:
Valid credentials discovered
Username: employee1
Password: Password123

Screenshot:
Passwpord_Spraying.png

---

### Step 4 — RDP Login

The attacker logs in using the discovered credentials.

Command executed:
xfreerdp3 /v:<WINDOWS_IP> /u:employee1 /p:Password123 /cert:ignore

Result:
Successful user-level access to the system.

Screenshot:
xfreerdp3-login+reconnaisence.png

---

### Step 5 — Post-Compromise Reconnaissance

The attacker performs basic system reconnaissance.

Commands executed:
whoami
hostname
systeminfo
ipconfig /all
net user

Result:
- User-level access confirmed
- System identity identified
- Local user structure enumerated

---

### Step 6 — Simulated Ransomware Impact

To simulate ransomware behavior without deploying real malware, files are renamed.

Command executed:
Get-ChildItem -File | Rename-Item -NewName { $_.Name + ".encrypted" }

A fake ransom note is left on the desktop demanding cryptocurrency payment.

Screenshot:
Ransomware+left_behind_message.png

Note:
No real malware is used.
This step demonstrates impact and business disruption only.

---

### Step 7 — Attacker Exit

The attacker logs out cleanly.

Command executed:
logoff

---

## SIEM Visibility During the Attack

Initial review of the Wazuh dashboard suggested no meaningful visibility.

After adjusting the time range to match the exact attack window:
- Approximately 1 hour of activity
- 1,387 events ingested
- Mostly authentication noise
- No alerts triggered
- No prioritization or correlation

Screenshot:
Noisy_wazuh.png

This demonstrates that high log volume without filtering causes malicious activity to disappear in noise.

---

## Post-Incident Improvements

Following the incident, partial security hardening is applied.

### Password Policy Improvements

- Minimum password length increased
- Password complexity enabled
- Password history enforced
- Maximum password age configured

Account lockout policy remains disabled.

Screenshot:
Hardened_password_policy.png

---

### Sysmon Deployment

Sysmon is installed using the SwiftOnSecurity configuration to improve endpoint visibility.

Verification performed:
Get-Service Sysmon64

Screenshot:
Sysmon_running.png

---

### Windows Audit Policy Improvements

Audit policy is partially strengthened.

- Logon success and failure enabled
- Process creation auditing enabled
- Policy change auditing enabled

Result:
- Attacker activity now generates telemetry
- Detection becomes possible
- Response capability is still missing

---

## Lessons Learned

Scenario 1 demonstrates that:
- Weak passwords enable trivial compromise
- Public RDP significantly increases risk
- Visibility without filtering is ineffective
- SIEM ingestion alone does not stop attacks
- SOC processes are as important as tooling

---

## End of Scenario 1 — Weak Endpoint

Snapshots taken:
scenario1_start
scenario1_end_windows
scenario1_end_kali
