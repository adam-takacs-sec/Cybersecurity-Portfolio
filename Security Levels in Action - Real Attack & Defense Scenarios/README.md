# 🛡️ Security Levels in Action  
**End-to-End Adversary Simulation & SOC Response Lab**

> ⚠️ **Lab-only project.**  
> All names, companies, users, IP addresses, domains, and data are entirely fictional and used strictly for educational and portfolio purposes.

---

## 📌 Project Overview

**Security Levels in Action** is a hands-on cybersecurity lab project that demonstrates how attacker behavior evolves across different security maturity levels — and how defensive capabilities (or the lack of them) directly impact outcomes.

The project follows a realistic, story-driven structure where I intentionally:
- start with a weak, exposed endpoint,
- progressively improve visibility,
- demonstrate why visibility alone is not enough,
- implement active detection and response,
- and finally harden the system based on real attack paths.

This project is designed to reflect **real SOC analyst work**, not CTF-style challenges.

---

## 🎯 Project Goals

- Demonstrate realistic attacker tradecraft (OSINT, credential abuse, recon)
- Show how weak configurations lead to compromise
- Highlight the difference between:
  - no visibility
  - visibility without response
  - active detection and response
- Practice SOC-style investigation and containment
- Translate incidents into concrete hardening actions
- Build a **portfolio-ready, explainable security narrative**

---

## 🧩 Project Structure

The project is divided into **five main parts**, each representing a different security maturity stage.

```
00 — OSINT Pre-Stage
01 — Scenario 1: Weak Endpoint
02 — Scenario 2: Improved Visibility, Weak Response
03 — Scenario 3: Detection & Response in Action
04 — Hardened Endpoint
```

Each part builds directly on the previous one.

---

## 🕵️ 00 — OSINT Pre-Stage

**Goal:** Simulate how an attacker prepares *before* any technical attack.

In this phase, I performed:
- Company profiling using public sources
- Employee profiling via social media
- Username pattern enumeration
- OSINT-derived password generation
- Phishing-based IP and environment fingerprinting

**Key takeaway:**  
No vulnerabilities were exploited — only publicly available information was used.

This stage feeds directly into Scenario 1 and Scenario 2.

---

## 🟥 Scenario 1 — Weak Endpoint

**Security state:**  
Minimal logging, weak password policy, no detection, no response.

**What happens:**
- Publicly exposed RDP
- Weak credentials
- No lockout policy
- No Sysmon
- Poor SIEM configuration

**Outcome:**
- Attacker gains access
- Performs reconnaissance
- Simulated ransomware impact
- Activity blends into log noise
- No alerts, no response

**Key lesson:**  
> Logs without structure and policy are useless.

---

## 🟧 Scenario 2 — Improved Visibility, Weak Response

**Security state:**  
Logging improved, but SOC processes still missing.

**What changes:**
- SIEM deployed (Wazuh)
- Windows Security logs ingested
- Sysmon installed
- Better audit policies

**What still fails:**
- No alert rules
- No thresholds
- No correlation
- No SOC workflows

**Outcome:**
- OSINT-based credential attack succeeds again
- Attacker is clearly visible in logs
- No alerts are triggered
- No response occurs

**Key lesson:**  
> Visibility alone does not equal security.

---

## 🟥 Scenario 3 — Detection & Response in Action

**Security state:**  
Active detection and response enabled.

**What changes:**
- Custom alerting rules
- Correlation logic
- SOC dashboards
- Defined response actions

**What happens:**
- Attacker reuses valid credentials
- Successful logon detected
- Recon activity observed
- SOC terminates the session
- IP is blocked
- Credentials are reset
- Attacker fails to re-enter even after IP change

**Outcome:**
- Initial access: Yes
- Impact: No
- Persistence: No

**Key lesson:**  
> Early detection + fast response breaks the attack chain.

---

## 🔒 Hardened Endpoint

**Security state:**  
Post-incident, production-ready hardening.

Based entirely on what was observed during the scenarios, I implemented:
- Enforced account lockout policy
- RDP exposure reduction
- Credential hygiene and session control
- PowerShell Script Block Logging
- Sysmon tuning
- Alert severity normalization
- SOC response workflows

**Outcome:**
- Repeat compromise using the same techniques is no longer viable
- Attacker dwell time is minimized
- Detection quality is significantly improved

**Key lesson:**  
> Hardening must be driven by real incidents, not theory.

---

## 🛠️ Tools & Technologies Used

- **Operating Systems**
  - Windows Server 2022 (endpoint-style deployment)
  - Kali Linux (attacker system)

- **Detection & Logging**
  - Wazuh SIEM
  - Windows Security Event Logs
  - Sysmon
  - PowerShell Operational Logs

- **Attack Techniques**
  - OSINT reconnaissance
  - Credential guessing (Hydra)
  - RDP abuse
  - Living-off-the-land reconnaissance

---

## 👨‍💻 Who This Project Is For

This project is intended for:
- Junior SOC Analysts
- Blue Team learners
- Cybersecurity students
- Hiring managers reviewing hands-on portfolios

It is **not**:
- A CTF walkthrough
- A theoretical whitepaper
- A vendor-specific tutorial

---

## 📎 How to Use This Repository

- Read scenarios in order
- Treat each scenario as a maturity step
- Review screenshots alongside explanations
- Focus on **why** things failed — not just *what* happened

---

## ✅ Final Takeaway

This project demonstrates a complete security lifecycle:

> **Attack → Visibility → Failure → Detection → Response → Hardening**

It shows that:
- Attacks don’t require exploits
- Logs don’t stop attackers
- Detection without response is ineffective
- Good SOC work is about speed, context, and discipline

---

## 📌 Status

✔ Completed  
✔ Fully documented  
✔ Portfolio-ready  

---

*Created as part of a hands-on cybersecurity learning and portfolio project.*
