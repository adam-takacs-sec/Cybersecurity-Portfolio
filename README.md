# 🔐 Adam — Practical Cybersecurity Portfolio

Hands-on cybersecurity projects focused on **real attacker behavior, real logs, and real SOC workflows**.

This repository documents my practical journey toward becoming a **Junior SOC Analyst / Cybersecurity Analyst**, with an emphasis on:
- detection and response,
- SIEM investigation,
- log-based analysis,
- and realistic adversary simulations.

Everything in this repository is built, tested, and documented in a live lab — not theoretical exercises.

---

## 🎯 Purpose of This Repository

The goal of this portfolio is to demonstrate that I can:

- think like an attacker **and** a defender
- generate realistic security telemetry
- investigate incidents using SIEM data
- understand why controls fail
- translate incidents into concrete hardening actions

This is meant to be **readable by both technical reviewers and hiring managers**, while still being technically accurate.

---

## 📦 Repository Structure (Overview)

This repository is organized into **four main areas**:

```
/log-bible
/attacks
/projects
/assets
```

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

This is a structured knowledge base built from hands-on lab work and real investigation needs.

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

The lab environment is built to be **realistic, minimal, and practical**, using tools and hardware that are commonly available outside of enterprise environments.

The goal is to demonstrate **security capabilities**, not hardware performance.

### 💻 Available Systems

- **MacBook Pro (M1)**
  - 8GB RAM
  - 256GB SSD
  - Used for:
    - Daily work
    - Documentation
    - Primary interface to Windows endpoints and the SIEM

- **Home PC**
  - Intel i5 2320 CPU
  - NVIDIA GTX 1660 Super
  - 8GB RAM
  - 500GB HDD
  - Used for:
    - Dedicated lab system
    - Used for attack simulation and testing activities

### ☁️ Cloud Infrastructure

- **Google Cloud Platform VMs**
  - Windows Server endpoints
  - Linux-based SIEM infrastructure (Wazuh)

Cloud systems provide:
- internet-exposed services
- realistic authentication traffic
- centralized log collection and correlation

### 🎯 Lab Focus

The focus of the lab is **not the underlying hardware**, but the outcomes it enables:

- real authentication events
- real endpoint telemetry
- SIEM ingestion and correlation
- attacker and SOC workflows across multiple systems

This environment shows that meaningful SOC and detection engineering work can be performed using **limited, realistic resources**.

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

- **Feature-complete** – core scenarios and documentation are finished
- **Ready for review** – structure, content, and evidence are in place
- **Actively maintained** – fixes, refinements, and improvements are ongoing
- **Continuously expanded** – supporting sections (e.g. attacks, log references, detections) will be extended over time

---

## 📬 Contact

If you are a recruiter, SOC analyst, or security professional and would like to discuss this work, feel free to reach out.
Email: adam.takacs.pr@proton.me

This repository represents how I think, investigate, and learn security — not just what tools I use.
