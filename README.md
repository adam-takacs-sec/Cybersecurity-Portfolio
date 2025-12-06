# 🔐 Ádám – Practical Cybersecurity Portfolio

Hands-on cybersecurity projects, SOC analysis work, log investigation, and attack simulations — built across a hybrid lab environment combining local ARM virtualization and cloud infrastructure.

This repository documents my practical journey toward becoming a Junior SOC Analyst / Cybersecurity Analyst, focusing on real-world skills, SIEM work, and detection engineering.

---

## 📦 Repository Structure (Overview)

This portfolio currently contains three main areas:

### **1. SOC Analyst Log Bible**
➡️ `/log-bible/`  
A comprehensive, from-scratch reference covering Windows logs, Sysmon, Linux logs, network telemetry, cloud audit logs, and EDR behavior.  
Designed as a practical guide for SOC analysts and blue teamers.

---

### **2. Attack Documentation**
➡️ `/attacks/`  
Focused, practical explanations of real attack techniques.  
Includes:
- Password Attacks (Brute Force, Spraying, Credential Abuse)

More attack topics will be added later as the portfolio evolves.

---

### **3. Practical Cybersecurity Projects**
➡️ `/projects/`  
This section will contain scenario-based projects demonstrating full attack & detection workflows (3–4 self-contained cases).  
Currently under construction — content will be added gradually.

---

## 🧱 Lab Environment (High-Level)

My cybersecurity lab is built as a **hybrid environment**:

### **Local (ARM Mac – UTM)**
- **Kali Linux (ARM)**  
  Used for attack simulation, scanning, enumeration, exploitation, and traffic generation.

### **Cloud (Google Cloud Platform – VM instances)**
- **Ubuntu Server (Wazuh SIEM + Docker services)**  
  Central log collection, analysis, detection engineering.
  
- **Windows Server (victim machine / AD / Sysmon)**  
  Produces real authentication, service, PowerShell, Sysmon, and attack telemetry.

This hybrid setup closely resembles a real enterprise SOC environment and enables:

- remote log ingestion  
- multi-host attack paths  
- real identity attacks  
- realistic detection pipelines  

---

## 🧭 Philosophy

I focus on **practical, reproducible, hands-on work**:

- real logs  
- real detections  
- real attack traffic  
- real SIEM pipelines  

No theory without demonstration.

---

## 📬 Contact

If you're a recruiter or cybersecurity professional interested in my work or experience, feel free to reach out.
