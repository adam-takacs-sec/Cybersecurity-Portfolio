# 🔥 Attacks

This directory contains practical, hands-on demonstrations of different cyber attack techniques used in real-world offensive operations and blue team detection workflows.  
Each attack is documented in a structured, SOC-oriented way, focusing on:

- How the attack works  
- Tools and methodology  
- What logs it generates  
- How a SOC analyst detects it  
- MITRE ATT&CK mapping  
- Realistic lab execution  

This section is part of the broader **Cybersecurity Portfolio**, where all attack simulations are performed in a controlled lab environment (Kali Linux attacker → cloud-based Windows & Linux targets → Wazuh SIEM).

---

## 📂 Current Attack Documentation

### 🔐 **Password Attacks**
➡️ [`password-attacks.md`](./password-attacks.md)

Covers:

- Brute-force  
- Dictionary attacks  
- Hybrid attacks  
- Password spraying  
- Credential stuffing  
- Wordlist generation (Crunch, CeWL)  
- Online brute-forcing tools (Hydra, Ncrack, Medusa, CME, Crowbar)  
- Offline cracking (John the Ripper)  
- SOC detection (Windows Events, Sysmon, Wazuh rules)  

---

## 🧭 Structure Philosophy

Each attack in this directory follows the same structure:

1. **Overview & Purpose**  
2. **Attack Prerequisites**  
3. **Execution Steps**  
4. **Tools Used**  
5. **Generated Logs (Windows, Sysmon, Linux, Network)**  
6. **MITRE ATT&CK Mapping**  
7. **Detection Opportunities (SIEM/EDR)**  
8. **Screenshots & Evidence**  
9. **Summary & Analyst Notes**

This makes every attack:

- easy to study  
- easy to expand  
- easy to reference during interviews  
- consistent across the entire portfolio  

---

## 🏗️ Future Additions  
(Will be added later — not listed until created)

- Web attacks  
- Network intrusion techniques  
- Lateral movement  
- Privilege escalation  
- Persistence mechanisms  

---

If you'd like to explore the next attack module or propose an improvement, feel free to open an issue or contact me.

