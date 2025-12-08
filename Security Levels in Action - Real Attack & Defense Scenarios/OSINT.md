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

**Screenshot placeholder:**  
`/assets/osint/company_facebook_page.jpg`

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

**Screenshot placeholders:**  
`/assets/osint/hermione_profile_about.jpg`  
`/assets/osint/hermione_profile_posts.jpg`

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

### 5.1 Phishing Email Screenshot  
`/assets/osint/phishing_email.jpg`

### 5.2 Fake Policy Page

The link directed her to a page styled as a corporate compliance update.  
Behind the scenes, the page logged her:

- external IP address  
- browser User-Agent  
- timestamp  

This information is crucial for validating the victim’s environment very early in the process.

**Screenshot placeholder:**  
`/assets/osint/phishing_policy_page.jpg`

### 5.3 Logged Result

A sample log entry from my tracking server:

