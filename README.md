# Ádám – Practical Cybersecurity Portfolio

Hands-on cybersecurity projects, SIEM implementations, incident response cases, log analysis, network investigation, and lab architectures — all built from scratch on an ARM-based Mac environment.

This repository documents my practical journey toward becoming a **Junior SOC Analyst / Cybersecurity Analyst**, focusing on real-world skills and demonstrable projects.

---

## 🔥 Goals

- Build a complete home cybersecurity lab (UTM + Docker + Linux)
- Deploy and configure Wazuh SIEM in an ARM environment
- Create custom detection rules, alerts, and log pipelines
- Process and analyze real attack traffic (pcaps, brute force, scanning)
- Write professional incident response case studies
- Understand MITRE ATT&CK techniques through hands-on examples

---

## 🧱 Lab Architecture

Full documentation here:  
➡️ [00_lab-architecture](./00_lab-architecture)

High-level overview:

- **Ubuntu Server (ARM)** – SIEM, Wazuh Manager, Docker services  
- **Ubuntu Desktop (ARM)** – SOC analyst workstation (Wireshark, analysis tools)  
- **Kali Linux (ARM)** – attack simulation, scanning, pcap generation  

---

## 📂 Project Index

### **1️⃣ Ubuntu Server Setup**
➡️ [01_ubuntu-server-setup](./01_ubuntu-server-setup)  
Base installation, hardening, Docker engine installation, and preparation for SIEM deployment.

---

### **2️⃣ Wazuh SIEM Installation (Docker, ARM)**
➡️ [02_wazuh-installation](./02_wazuh-installation)  
Wazuh Manager + Indexer + Dashboard deployment in an ARM-native Docker environment.

---

### **3️⃣ Custom Wazuh Rules**
➡️ [03_wazuh-custom-rules](./03_wazuh-custom-rules)  
Handwritten detection rules, alert mappings, test cases, and log generation.

---

### **4️⃣ Sysmon for Linux (ARM)**
➡️ [04_sysmon-linux](./04_sysmon-linux)  
Sysmon events → Wazuh ingestion → custom Sigma/Sysmon-based detection.

---

### **5️⃣ Incident Response Case Studies**
➡️ [05_incident-response-cases](./05_incident-response-cases)  
Realistic mini-investigations, log analysis, and MITRE ATT&CK mapping.

---

## 🖼️ Screenshots

All screenshots used in reports, detections, SIEM dashboards, and lab steps:  
➡️ [screenshots](./screenshots)

---

## 📑 Documentation Style

Every project contains:

- Clear overview  
- Step-by-step technical execution  
- Commands used  
- Architecture diagrams (when needed)  
- Screenshots (key evidence only)  
- Detection logic  
- Results + findings  
- MITRE ATT&CK technique mapping  
- Short reflection  

Everything is optimized for **interview readability**.

---

## 📬 Contact

If you're a recruiter or cybersecurity professional interested in my work, feel free to reach out.

