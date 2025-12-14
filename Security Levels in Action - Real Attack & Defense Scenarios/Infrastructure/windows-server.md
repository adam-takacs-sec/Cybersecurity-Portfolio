# Windows Server – Target Endpoint System

## Purpose
This system represents the **primary attacked endpoint** in the scenario.

Although Windows Server is used as the operating system, this machine
**does not function as an infrastructure server**.
It represents a **user workstation exposed directly to the internet**,
intentionally deployed with weak security controls to demonstrate
realistic endpoint compromise and SIEM telemetry generation.

The operating system choice is driven by **cloud platform constraints**,
not by architectural intent.

---

## Platform & Hardware
- Cloud provider: Google Cloud Platform (GCP)
- Region / Zone: europe-west4-b
- Machine type: e2-standard-4
  - vCPU: 4
  - Memory: 16 GB
- Storage: 80 GB Persistent Disk

---

## Operating System Choice Rationale

### Why Windows Server instead of Windows 10?
Google Cloud Platform does not provide native Windows 10 images and deploying
Windows 10 in GCP requires complex workarounds and licensing overhead.

Windows Server is therefore used as a **technical substitute** for a Windows
endpoint, while being treated **purely as a user machine**, not as a server.

No server roles or enterprise services are installed.

---

### Why Kali Linux on bare metal?
Kali Linux is used as the attacker system and runs on bare metal due to:

- No official Kali Linux images available in GCP
- Running Kali inside UTM on Apple Silicon introduces:
  - ARM compatibility limitations
  - Network isolation issues
  - Host firewall interference
  - Reduced realism for attack tooling

Bare metal deployment ensures:
- Full tool compatibility
- Stable networking
- Realistic attacker behavior

---

## Installation
- OS: Windows Server 2022
- Deployment type: Cloud VM
- Usage model: **Standalone endpoint**
- Initial access method: RDP (publicly accessible)

No additional Windows roles or features were installed.
The system remains in a default, “vanilla” state.

---

## User Accounts

The system simulates a real user environment.

- Local Administrator account:
  - Used only for system setup
  - Not used during the attack
- Standard user account:
  - Represents the victim employee
  - Primary attack target
  - Used for interactive logons

Attacks are performed **against the standard user context**, not directly
against the administrator account.

---

## Network Access
- RDP (3389/TCP) – user access and attack surface
- Wazuh Agent communication:
  - 1514/TCP
  - 1515/TCP

No IP filtering or network hardening applied.

---

## Firewall Configuration (GCP)

Ingress rules (intentionally permissive):

- TCP 3389 – RDP
- TCP 1514 – Wazuh agent data channel
- TCP 1515 – Wazuh agent enrollment

Source range: `0.0.0.0/0`  
Target: Windows endpoint VM

This configuration reflects a weak perimeter posture.

---

## Wazuh Agent Integration
The endpoint is enrolled as a Wazuh agent to provide baseline visibility.

- Official Wazuh Windows agent installed
- Enrollment against Wazuh SIEM internal IP
- Default configuration used
- No additional log sources enabled

Telemetry flow verified.

### Wazuh Agent Active (Dashboard)
![Windows Server Desktop](../../Assets/Infrastructure/windows-server/wazuh-dashboard.png)

---

## Authentication Configuration (Baseline)

Authentication controls are intentionally weak.

- Password complexity: Enabled (default)
- Minimum password length: Low
- Account lockout policy: Ineffective
- RDP exposed publicly
- No MFA
- No credential hardening

This reflects a common real-world misconfiguration.

### Password Policy
![Password Policy](../../Assets/Infrastructure/windows-server/password-policy.png)

### Lockout Policy
![Wazuh Agent Active](../../Assets/Infrastructure/windows-server/lockout-policy.png)

---

## Post-install Verification

- Successful RDP login as standard user
- Wazuh agent connected and reporting
- Windows Security Event Logs generated
- No additional security hardening

---

## Configuration Scope

The following controls are intentionally **not enabled**:

- Sysmon
- Advanced audit policies
- PowerShell logging
- Credential Guard
- Endpoint protection tuning

This allows successful compromise while still producing basic telemetry.

---

## Security Posture Summary

- Publicly exposed endpoint
- Weak authentication
- Minimal logging
- No active defense mechanisms

This system is intentionally vulnerable to demonstrate
how endpoint misconfiguration leads to compromise.

---

## Status
Windows endpoint system is operational and ready for
attack execution and SIEM-based analysis.
