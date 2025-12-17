# 🔐 Password Attack Toolkit – Cheat Sheet

This document summarizes the most important **password attack methods** and the related **tools**.

**Included tools:**  
Hydra, Ncrack, CrackMapExec, Crowbar, Medusa, Crunch, CeWL, John the Ripper

Intended for **lab / pentest / SOC learning** purposes.

> ⚠️ **Warning:** All examples are for use **only in your own lab environment**, against systems you own or are explicitly authorized to test.  
> Running password attacks against third-party systems is illegal.

---

## 1. Password Attack Types

### 1.1 Brute-force attack
- **What does it do?**  
  Tries **every possible character combination** in the search space (e.g. `aaaaaa` → `zzzzzz`).
- **Pros:**  
  Guaranteed to find the password if it exists in the defined keyspace.
- **Cons:**  
  Becomes extremely slow as password length and character set size increase.

### 1.2 Dictionary attack
- **What does it do?**  
  Uses a pre-defined password list (wordlist), e.g. `rockyou.txt`.
- **Pros:**  
  Faster, since it tests **real, commonly used** passwords.
- **Cons:**  
  If the password is not in the wordlist, it will never be found.

### 1.3 Hybrid attack
- **What does it do?**  
  Combines a wordlist + variations.  
  Example base word: `Princess` → `Princess1`, `Princess123`, `Princess74!`, `pr1nc3ss`.
- **When to use:**  
  When you know some **personal information** about the target:
  - pet name  
  - nickname  
  - favorite game, band, etc.

### 1.4 Rule-based attack
- **What does it do?**  
  Applies **rules** to transform base words into password candidates:
  - capitalize first letter  
  - add numbers at the end  
  - character substitutions: `a → @`, `o → 0`, `i → 1`
- **Pros:**  
  Models very well how people create “smart” but predictable passwords.

### 1.5 Credential stuffing
- **What does it do?**  
  Uses **real leaked username + password pairs** and tries them on other services.
- **Example:**  
  A website gets breached, and attackers try the same credentials on:
  - email  
  - VPN  
  - Office 365  
  - corporate portals  

### 1.6 Password spraying
- **What does it do?**  
  Tries **the same password** on **many different accounts**, e.g. `Welcome2024!`.
- **Goal:**  
  Avoid account lockout by:
  - using a few common passwords
  - across many users  
  - with long time delays between attempts.

---

## 2. Password Attack Tools – Overview

| Tool               | Description                                               | Typical Protocols                          | Attack Types                         |
|--------------------|-----------------------------------------------------------|--------------------------------------------|--------------------------------------|
| **Hydra**          | Fast, multi-protocol online brute-forcer                  | SSH(22), FTP(21), SMB(445), RDP(3389), HTTP | brute-force, dictionary, hybrid      |
| **Ncrack**         | Enterprise-grade brute-forcer by the Nmap team            | SSH, RDP, VNC, FTP, Telnet                 | brute-force, slow brute-force        |
| **CrackMapExec**   | Windows network & credential checking framework           | SMB(445), RDP(3389), WinRM(5985/5986)      | credential stuffing, password spray  |
| **Crowbar**        | Simple RDP/VNC brute forcer                               | RDP(3389), VNC(5900)                       | brute-force                          |
| **Medusa**         | Modular, parallel brute-forcer                            | SSH, SMB, FTP, HTTP, SQL, POP3, IMAP       | brute-force                          |
| **Crunch**         | Mask-based password generator                             | –                                          | hybrid / brute-force preparation     |
| **CeWL**           | Wordlist generator from website content (OSINT-driven)    | HTTP/HTTPS                                  | dictionary, hybrid                   |
| **John the Ripper**| Offline password/hash cracker                             | – (hash input)                             | offline dictionary, rule-based       |

---

## 3. Hydra – Fast Multi-Protocol Brute Forcer

### 3.1 What does it do?

- Performs brute-force or dictionary attacks against various network services.
- Supports multi-threading (`-t`) → runs many attempts in parallel.
- Modular structure: `ssh://`, `rdp://`, `smb://`, etc.

### 3.2 Common protocols & ports

- SSH – 22  
- FTP – 21  
- Telnet – 23  
- HTTP/HTTPS form login – 80 / 443  
- SMB – 445  
- RDP – 3389  
- VNC – 5900  
- POP3 – 110  
- IMAP – 143  

### 3.3 Basic commands

**Test a single password against SSH:**

```bash
hydra -l admin -p Password1! ssh://10.10.10.10
```

**Wordlist attack against SSH:**

```bash
hydra -l admin -P rockyou.txt -t 4 -W 3 ssh://10.10.10.10
```

**RDP brute-force (slow & cautious):**

```bash
hydra -l admin -P passwords.txt -t 1 -W 2 rdp://34.34.20.16
```

### 3.4 Parameter explanation

| Parameter         | Meaning                                                 |
|-------------------|---------------------------------------------------------|
| `-l admin`        | Single username                                         |
| `-L users.txt`    | Username list                                           |
| `-p Password1!`   | Single password                                         |
| `-P passlist.txt` | Password list (wordlist)                                |
| `-t 4`            | Number of threads (more threads → faster, more noisy)   |
| `-W 3`            | Wait 3 seconds between failed attempts                  |
| `ssh://`          | SSH module                                              |
| `rdp://`          | RDP module                                              |
| `-s 2222`         | Custom port (e.g. non-default SSH port)                 |

### 3.5 Recommended usage

- Fast multiprotocol brute-forcer for small lab environments.
- Good for generating:
  - many **4625 (Failed Logon)** events in Windows logs  
  - realistic brute-force traffic for SIEM/Wazuh training.

---

## 4. Ncrack – Reliable Enterprise Brute Forcer

### 4.1 What does it do?

- Built by the **Nmap** team.
- Optimized for large networks and **stable brute-force** operations.
- Handles latency and timeouts more gracefully than Hydra.

### 4.2 Common protocols

- SSH – 22  
- RDP – 3389  
- VNC – 5900  
- FTP – 21  
- Telnet – 23  

### 4.3 Basic commands

**SSH brute-force:**

```bash
ncrack -p 22 --user admin -P rockyou.txt 10.10.10.10
```

**RDP brute-force:**

```bash
ncrack -p 3389 --user admin -P passwords.txt 34.34.20.16
```

**Slower, more cautious mode (delay):**

```bash
ncrack -p 3389 --user admin -P passwords.txt --connection-limit 1 --delay 5s 34.34.20.16
```

### 4.4 Recommended usage

- For larger wordlists and **“no-lockout”** style testing.
- Stable RDP brute-force when Hydra’s module is unreliable.
- Good for enterprise-style attack simulations.

---

## 5. CrackMapExec (CME) – Credential Validation & Lateral Movement

### 5.1 What does it do?

Windows / AD-focused tool that does **much more than brute force**:

- Credential validation (check if username+password works)  
- SMB / WinRM / RDP access check  
- Domain / AD enumeration  
- Often used as part of lateral movement playbooks

### 5.2 Ports

- SMB – 445  
- RDP – 3389  
- WinRM – 5985 (HTTP) / 5986 (HTTPS)  

### 5.3 Basic commands

**SMB credential check with a single password:**

```bash
crackmapexec smb 10.10.10.10 -u admin -p Password1!
```

**SMB brute-force for a single user:**

```bash
crackmapexec smb 10.10.10.10 -u admin -P passwords.txt
```

**RDP login check:**

```bash
crackmapexec rdp 10.10.10.10 -u admin -p Password1!
```

**WinRM access check (PowerShell remoting):**

```bash
crackmapexec winrm 10.10.10.10 -u admin -p Password1!
```

### 5.4 Recommended usage

- In Windows environments, this is **far more useful** than generic Hydra.
- Perfect for simulating:
  - credential stuffing  
  - password spraying  
  - lateral movement scenarios  

- From a SOC perspective:
  - generates 4625/4624 events  
  - WinRM-related PowerShell logs (4104, etc.)  
  - SMB connections (Sysmon Event ID 3)

---

## 6. Crowbar – Simple RDP/VNC Brute Forcer

### 6.1 What does it do?

- Focused brute-forcer for **RDP and VNC**.
- Fewer features, but simple and stable for these protocols.

### 6.2 Ports

- RDP – 3389  
- VNC – 5900  

### 6.3 Basic commands

**RDP brute-force:**

```bash
crowbar -b rdp -s 34.34.20.16/32 -u admin -C passwords.txt
```

**VNC brute-force:**

```bash
crowbar -b vnc -s 10.10.10.10/32 -u admin -C vnc_pass.txt
```

### 6.4 Recommended usage

- When you specifically want to target RDP, and Hydra is unstable.
- Great for simple lab scenarios.

---

## 7. Medusa – Modular Fast Brute Forcer

### 7.1 What does it do?

- High-performance, modular brute-force tool.
- Supports many different services.

### 7.2 Common protocols

- SSH  
- FTP  
- HTTP  
- SMB  
- POP3 / IMAP  
- MSSQL / MySQL  

### 7.3 Basic command

```bash
medusa -h 10.10.10.10 -u admin -P passlist.txt -M ssh
```

| Parameter           | Meaning                     |
|---------------------|----------------------------|
| `-h 10.10.10.10`    | target host                |
| `-u admin`          | username                   |
| `-P passlist.txt`   | password list              |
| `-M ssh`            | module / protocol (ssh/http/…) |

### 7.4 Recommended usage

- Good alternative when other tools are unstable.
- Wide protocol support → flexible for lab work.

---

## 8. Crunch – Advanced Mask-Based Password Generator

### 8.1 What does it do?

- Does **not perform brute force itself**.  
  Instead, it **generates wordlists** based on:
  - fixed length  
  - character sets  
  - custom masks / patterns  

### 8.2 Mask symbols for `-t` (pattern)

| Symbol | Meaning              |
|--------|----------------------|
| `@`    | lowercase letters    |
| `,`    | uppercase letters    |
| `%`    | digits (0–9)         |
| `^`    | special characters   |

### 8.3 Examples

**1) `Princess` + two digits (`Princess00`–`Princess99`):**

```bash
crunch 10 10 -t Princess%% -o princess.txt
```

- 10 = minimum length  
- 10 = maximum length  
- `Princess%%` = 8 fixed chars + 2 digits

**2) 8-character lowercase only wordlist:**

```bash
crunch 8 8 abcdefghijklmnopqrstuvwxyz -o 8lower.txt
```

**3) Name + 4 special characters (mask):**

```bash
crunch 9 9 -t Adam^^^^ -o adam_symbols.txt
```

### 8.4 Recommended usage

- To generate **targeted wordlists** for hybrid attacks (keyword + variations).
- Much more efficient than huge generic lists.
- Output `.txt` can be used directly by Hydra/Ncrack/CME.

---

## 9. CeWL – OSINT-Based Wordlist Generator

### 9.1 What does it do?

- Crawls a website and:
  - extracts words over a certain length (e.g. 5+ chars)
  - collects names, project names, etc.
  - outputs them as a wordlist

### 9.2 Why is it useful?

- Real attackers love using **company-related / project-related** phrases as passwords:
  - `Acme2024!`  
  - `ProjectX!`  
  - `AlphaTeam01`  

### 9.3 Examples

**Basic wordlist generation:**

```bash
cewl https://target-site.com -w wordlist.txt
```

**Deeper crawl (more links followed):**

```bash
cewl https://target-site.com -d 3 -w deep.txt
```

**Include numbers (date-like patterns):**

```bash
cewl https://target-site.com --with-numbers -w words_numbers.txt
```

### 9.4 Recommended usage

- For OSINT-based attack preparation.
- When you know the company/project naming style.
- Combine with Crunch:  
  CeWL → `wordlist.txt` → Crunch masks → Hydra/CME input.

---

## 10. John the Ripper – Offline Password Cracker

### 10.1 What does it do?

- Does **not** attack over the network.  
  It cracks **password hashes offline**, for example:
  - `/etc/shadow` (Linux)  
  - Windows NTLM/SAM hashes  
  - ZIP/RAR passwords  
  - and more  

### 10.2 Main attack modes

- Dictionary attack → `--wordlist=...`  
- Rule-based attack → dictionary + transformation rules  
- Incremental → full brute-force over a keyspace  

### 10.3 Examples

**Crack hashes using a wordlist:**

```bash
john hashes.txt --wordlist=rockyou.txt
```

**Resume a cracking session:**

```bash
john --restore
```

**Show cracked passwords:**

```bash
john --show hashes.txt
```

**Incremental mode (slow, full brute-force):**

```bash
john --incremental=All hashes.txt
```

### 10.4 Recommended usage

- After you have obtained hashes (e.g. in a lab from Windows SAM / NTDS.dit).
- Much faster and stealthier than online brute-force.
- From SOC’s POV:
  - watch for `reg save`, `ntdsutil`, `vssadmin` usage → indicators of hash dumping.

---

## 11. SOC Perspective – What Logs Do These Attacks Generate?

### 11.1 Windows Security Logs – Event IDs

- **4625** – Failed logon  
- **4624** – Successful logon  
- **4648** – Logon with explicit credentials  
- **4776, 4768, 4769, 4771** – Kerberos/NTLM authentication events (AD environments)

### 11.2 Sysmon

- **Event ID 1** – Process creation (e.g. `hydra.exe`, `ncrack`, etc.)  
- **Event ID 3** – Network connection (many login attempts from one source)  
- **Event ID 10** – Process access (if memory dumping is involved)

### 11.3 Wazuh / SIEM Views

- Authentication failure correlation rules  
- RDP brute-force detection (lots of 4625 in a short period)  
- WinRM anomalous login detection  
- PowerShell suspicious activity (reverse shell, encoded command)
