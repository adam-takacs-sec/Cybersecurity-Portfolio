# 🕵️ 00 — OSINT Pre-Stage  
**Security Levels in Action – Pre-Attack Information Gathering**

> ⚠️ Lab-only. All names, companies, accounts, domains and data are entirely fictional and used strictly for educational purposes.

Before initiating Scenario 1 and Scenario 2, I performed a complete OSINT reconnaissance phase.  
My goal was to gather enough publicly available information to build:

- a company profile  
- a targeted employee profile  
- internal username candidates  
- password seeds based on personal data  
- the user’s external IP address via phishing and tracking  

This OSINT stage simulates how a real attacker prepares before attempting any intrusion.

---

## 1. Company Profiling — Hogwarts Logistics Ltd.

I began by examining the publicly visible footprint of the target organization.

**Company name:** Hogwarts Logistics Ltd.  
**Industry:** Logistics / warehousing / delivery  
**Location:** London, United Kingdom  
**Public website:** `hogwartslogistics.com`  
**Public email:** `info@hogwartslogistics.com`

I visited the company’s Facebook page, where I could observe:

- branding and logo  
- images of the warehouse and office building  
- interactions from users who appear to be employees  
- recent marketing posts  
- visible names and likes connected to real profiles  

These details confirmed that the company has an active public presence—an ideal OSINT starting point.

![Hogwarts Logistics Ltd. Facebook page](/Assets/OSINT/Hogwarts%20Logistics%20Ltd.%20Home%20page.jpg)
*Figure 1 — Public Facebook page of Hogwarts Logistics Ltd. This page provides initial OSINT indicators such as branding, employee interactions, public posts, and visible company infrastructure.*

---

## 2. Employee Profiling — Hermione Granger

Next, I searched for employees who publicly interact with the company online.  
One profile stood out immediately: **Hermione Granger**.

### 2.1 Public Profile Information

From her Facebook profile:

- **Works at:** Hogwarts Logistics Ltd.  
- **Lives in:** London  
- **Education:** Hogwarts School of Witchcraft and Wizardry  
- **Relationship:** In a relationship with Ron Weasley  

### 2.2 Basic Info (About Page)

Her “Contact and basic info” section revealed:

- **Gender:** Female  
- **Birthdate:** September 19  
- **Birth year:** 1999  

So I recorded the following OSINT identifiers:

- `BirthYear = 1999`  
- `DayMonth = 0919`

### 2.3 Public Posts — Pet Information

One of her posts was especially valuable:

> “Crookshanks just turned 5 today! 🧡  
> Still the fluffiest little troublemaker.”

From this I learned:

- **Pet name:** Crookshanks  
- **Pet age mentioned:** 5 years  
- The pet has emotional significance  
- She posts personal milestones publicly  

Pet names and birth years are extremely common password components.

![Hermione Granger – Facebook Main Profile](/Assets/OSINT/Hermione%20Granger%20Posts%20page.jpg)
*Figure 2 — Hermione Granger’s public Facebook profile. This confirms her employment at Hogwarts Logistics Ltd. and exposes additional personal identifiers valuable for OSINT.*

![Hermione Granger – Facebook About Page](/Assets/OSINT/Hermione%20Granger%20About%20page.jpg)
*Figure 3 — Hermione’s “About” page revealing her birthdate (19 September) and birth year (1999). Birth-related information is a common element in user-created passwords.*

---

## 3. Username Format Enumeration

Based on the company’s public email format (`info@hogwartslogistics.com`), I inferred a likely employee pattern:

`firstname.lastname@companydomain`

Therefore, Hermione’s assumed corporate email is:

- **`hermione.granger@hogwartslogistics.com`**

From this pattern I generated several internal username candidates:

- `hermione.granger`  
- `hgranger`  
- `grangerh`  
- `hermione.g`  
- `h.granger`  

These values were saved into a username wordlist to be used in **Scenario 2 – OSINT-based credential guessing**.

**Screenshot placeholder:**  
`/assets/osint/usernames_file.jpg`

---

## 4. Password Seed Collection (OSINT-Based)

Using the personal data I discovered, I compiled a list of meaningful password seeds.

### 4.1 Raw Seeds Extracted

- `Hermione`  
- `Granger`  
- `HG`  
- `1999`  
- `0919`  
- `Crookshanks`  
- `5`  
- `Weasley`  
- `Hogwarts` / `HogwartsLogistics`

### 4.2 Common User Patterns Applied

To generate realistic password candidates, I applied common transformation habits:

- appending birth years → `1999`  
- appending personal numbers → `5`, `19`  
- appending dates → `0919`  
- capitalized variations  
- adding simple suffixes → `!`, `?`, `_`, `#`  
- light leetspeak → `H0gwarts`  
- concatenations → `CrookshanksHermione`, etc.

These transformations allowed me to produce a focused but highly effective custom password list.

---

## 5. Phishing Pretext & External IP Capture

Since I required Hermione’s external IP address for future stages, I crafted a realistic but generic pretext:

**Email subject:** `Mandatory Security Policy Update – Action Required by 6 December`  
**Sender (spoofed):** `hr-compliance@hogwartslogistic.com`  
**Call to action:** “Review Updated Policy”

The goal of this phishing email was not credential theft—it was simply to get Hermione to click a link that would load a tracking page on my server.

![Phishing Email – Security Policy Update Pretext](/Assets/OSINT/Phishing%20email%20OSINT.jpg)
*Figure 5 — The phishing email I sent to Hermione, disguised as a mandatory “Security & Data Handling Policy Update.” The purpose was to deliver a tracking link to capture her external IP address.*

### 5.2 Fake Policy Page

The link directed her to a page styled as a corporate compliance update.  
Behind the scenes, the page logged her:

- external IP address  
- browser User-Agent  
- timestamp  

This information is crucial for validating the victim’s environment very early in the process.

![Phishing Landing Page – Fake Policy Acknowledgement](/Assets/OSINT/Phishing%20email%20acknowledge.png)
*Figure 6 — The phishing landing page presented after clicking the email link. Although it appears to display a legitimate policy acknowledgement, its primary purpose is to log the victim’s IP address and User-Agent.*

### 5.2.1 Tracking Server Code (Policy Page + Logging)

> This server does **not** deliver malware or exploits.  
> Its sole purpose is early-stage validation and reconnaissance (IP + User-Agent + timestamp).

**File:** `policy_server.py`

~~~~python
from flask import Flask, request, render_template_string, send_file
from datetime import datetime
import os

LOGO_PATH = "/tmp/policy_static/hogwarts_orange.png"

app = Flask(__name__)

POLICY_HTML = """
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Hogwarts Logistics – Policy Update</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f3f4f6;
            color: #1f2937;
            padding: 40px;
        }
        .wrapper {
            max-width: 720px;
            margin: 0 auto;
            background: white;
            border-radius: 12px;
            padding: 32px;
            box-shadow: 0 4px 16px rgba(0,0,0,0.05);
        }

        /* Acknowledgement Box */
        .acknowledge-box {
            display: flex;
            align-items: flex-start;
            gap: 12px;
            background: #ecfdf5;
            border: 1px solid #a7f3d0;
            padding: 16px;
            border-radius: 8px;
            margin-top: 12px;
            margin-bottom: 24px;
        }
        .ack-icon {
            font-size: 22px;
            font-weight: bold;
            color: #059669;
            line-height: 1;
        }
        .ack-head {
            font-size: 16px;
            font-weight: bold;
            color: #065f46;
            margin-bottom: 2px;
        }
        .ack-text {
            font-size: 14px;
            color: #047857;
            line-height: 1.4;
        }

        h1 {
            font-size: 24px;
            margin-bottom: 12px;
        }
        h2 {
            margin-top: 28px;
            font-size: 18px;
        }
        p {
            line-height: 1.45;
            margin-bottom: 14px;
        }
        ul {
            margin-left: 20px;
            margin-bottom: 12px;
        }
        .small {
            font-size: 13px;
            color: #4b5563;
        }
    </style>
</head>
<body>

<div class="wrapper">

    <div style="display:flex; align-items:center; gap:14px; margin-bottom:20px;">
        <img src="/logo" alt="Hogwarts Logistics Logo"
             style="height:48px; width:auto; display:block; border-radius:8px;">
        <div>
            <div style="font-size:18px; font-weight:bold;">Hogwarts Logistics Ltd.</div>
            <div style="font-size:13px; color:#6b7280;">Security & Data Handling Notice</div>
        </div>
    </div>

    <h1>Security & Data Handling Policy Update</h1>

    <p>
        As part of our annual compliance review, the following areas have been updated:
    </p>

    <ul>
        <li>Password and credential requirements</li>
        <li>Remote access rules during the holiday period</li>
        <li>Data handling and classification guidelines</li>
        <li>Incident reporting workflow</li>
    </ul>

    <h2>Your acknowledgement</h2>

    <div class="acknowledge-box">
        <div class="ack-icon">✔</div>
        <div>
            <div class="ack-head">Acknowledgement Recorded</div>
            <div class="ack-text">
                By viewing this page, you confirm that you have received the latest version of the
                Security & Data Handling Policy for Hogwarts Logistics Ltd.
            </div>
        </div>
    </div>

    <p class="small">
        If you have any questions, please contact the HR Compliance team.
    </p>

</div>
</body>
</html>
"""

# Serves the logo used by the HTML page for realistic branding
@app.route("/logo")
def logo():
    if not os.path.exists(LOGO_PATH):
        return "Logo not found", 404
    return send_file(LOGO_PATH, mimetype="image/png")

# Main endpoint embedded in the phishing email
# Logs client IP + User-Agent + timestamp, then renders the policy page normally
@app.route("/policy-update")
def policy_update():
    ip = request.remote_addr
    ua = request.headers.get("User-Agent", "-")
    ts = datetime.utcnow().isoformat()

    log_line = f"{ts} | IP={ip} | UA={ua}\n"
    print(log_line.strip())

    with open("policy_clicks.log", "a") as f:
        f.write(log_line)

    return render_template_string(POLICY_HTML)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
~~~~

---

### 5.3 Logged Result

A sample log entry from my tracking server:

Victim IP: 89.xxx.xxx.xxx  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)  
Timestamp: 2025-12-04 09:52:31

This confirmed that Hermione clicked the link and provided me with the necessary network fingerprint.

**Screenshot:**  
`/assets/osint/ip_capture_log.jpg`

---

## 6. Final OSINT-Based Password List

Using all seeds and transformation rules, I generated a targeted password list (`osint_passwords.txt`).  
This list is intentionally compact, containing only passwords derived from Hermione’s real personal data.

Example entries:

- Crookshanks  
- Crookshanks1  
- Crookshanks5  
- Crookshanks19  
- Crookshanks2019  
- Crookshanks2019!  
- Crookshanks0919  
- Crookshanks0919!  
- Crookshanks#5  
- Crooks5!  
- Hermione1999  
- Hermione_1999  
- Hermione1999!  
- Hermione0919  
- Hermione0919!  
- Hermione#99  
- HG1999!  
- HG0919!  
- Hogwarts1999!  
- H0gwarts1999!  
- Weasley99!  
- CrookshanksHermione99  
- Crookshanks_Hogwarts19  
- HL_Crook2019!  
- Crookshanks5Years!  
- CrookshanksTurns5!  

This list will be used in **Scenario 2 – OSINT-based username & password guessing**.

---

## ✅ End of OSINT Pre-Stage

At this point I have collected:

- a complete company profile  
- a detailed employee profile  
- internal username candidates  
- a targeted OSINT-derived password list  
- the victim’s external IP and User-Agent  

With all reconnaissance completed, I can now proceed to:

- **Scenario 1 – Generic Brute Force (baseline)**  
- **Scenario 2 – OSINT-Based Credential Guessing**
