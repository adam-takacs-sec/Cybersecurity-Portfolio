
Each section serves a different purpose.

---

## 📘 1. SOC Analyst Log Bible  
➡️ `/log-bible/`

A comprehensive, from-scratch reference created during my learning process.

Contents include:
- Windows Security Event IDs (authentication, account management, process creation)
- Sysmon telemetry and use cases
- PowerShell logging (limitations and improvements)
- Linux authentication and system logs
- Network and authentication concepts
- SIEM investigation notes and correlations

This is **not copied documentation** — it is a structured knowledge base built from hands-on lab work and real investigation needs.

---

## ⚔️ 2. Attack Documentation  
➡️ `/attacks/`

Focused documentation of **specific attack techniques**, written from an attacker’s perspective but analyzed from a SOC viewpoint.

Current topics include:
- Password brute force
- Password spraying
- Credential abuse
- OSINT-based credential attacks

Each attack description explains:
- why it works,
- what logs it generates,
- and how it should be detected.

This section will expand over time.

---

## 🧪 3. Practical Cybersecurity Projects  
➡️ `/projects/`

Scenario-based, end-to-end projects demonstrating full **attack → detection → response → hardening** workflows.

The main project currently documented here is:

### **Security Levels in Action**
A multi-stage lab that includes:
- OSINT Pre-Stage
- Scenario 1 — Weak Endpoint
- Scenario 2 — Improved Visibility, Weak Response
- Scenario 3 — Detection & Response in Action
- Hardened Endpoint (Post-Incident)

Each scenario builds logically on the previous one and reflects realistic SOC operations rather than artificial challenges.

---

## 🖼️ 4. Assets  
➡️ `/assets/`

This directory contains **all screenshots and visual evidence** used across the repository, including:
- attacker-side execution
- Windows endpoint activity
- Wazuh SIEM dashboards
- Discover queries
- authentication events
- process execution evidence

Keeping screenshots centralized ensures:
- clean documentation
- easy reuse
- consistent references across scenarios

---

## 🧱 Lab Environment (High-Level Overview)

The lab environment is intentionally simple at a high level, while still realistic.

It consists of:
- a **Mac** used for daily work and ARM-based virtualization,
- a **PC** used for additional local testing,
- **Google Cloud Platform VMs** hosting:
  - Windows Server endpoints
  - Linux-based SIEM infrastructure (Wazuh)

The focus is **not on the hardware**, but on the security outcomes:
- real authentication traffic
- real endpoint telemetry
- real SIEM ingestion and correlation

---

## 🧭 Philosophy

My approach is strictly **hands-on and evidence-driven**.

I focus on:
- real logs instead of assumptions
- detection logic instead of dashboards only
- attacker behavior instead of checklists
- response actions instead of passive visibility

If something is detected, it must be explainable.
If something fails, the reason must be clear.
If something is hardened, it must directly address an observed weakness.

---

## 📈 Current Status

- Actively maintained
- Continuously expanded
- Portfolio-focused
- Junior SOC Analyst oriented

---

## 📬 Contact

If you are a recruiter, SOC analyst, or security professional and would like to discuss this work, feel free to reach out.
Email: adam.takacs.pr@proton.me

This repository represents how I think, investigate, and learn security — not just what tools I use.
