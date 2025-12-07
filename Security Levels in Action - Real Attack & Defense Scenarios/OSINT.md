# 🕵️ 00 — OSINT Pre-Stage  
**Security Levels in Action – Pre-Attack Information Gathering**

> ⚠️ Lab-only. All names, companies and user data are fictional.  
> This stage is performed **before Scenario 1, 2, 3 and 4**, and prepares all intelligence the attacker will use.

This document simulates realistic OSINT techniques used to gather information about a company, an employee, and their environment — **before any intrusion attempts**.

The goal is to build:
- a **company profile**  
- an **employee profile**  
- **username seeds**  
- **password seeds**  
- an **external IP capture** (user workstation or corporate gateway IP)  

---

# 1. Company Profiling (Fictional)

The attacker identifies a target organization:

- Company name: \`Northwind Logistics Kft\`  
- Industry: Logistics / shipping  
- Headquarters: Budapest  
- Public website: \`northwindlogistics.example\`  
- Public social media pages (fictional): Facebook, LinkedIn  

The attacker reviews:
- employee posts  
- company news  
- visible employees in photos  
- job descriptions (technology hints)  

---

# 2. Employee Profiling (OSINT)

From public social media information, the attacker identifies one employee:

### Target Employee (fictional)
- Name: \`Anna Kovács\`  
- Role: \`Customer Support Specialist\`  
- Department: Customer Operations  
- Email format at company: \`firstname.lastname@northwindlogistics.example\`  
- Reconstructed email: \`anna.kovacs@northwindlogistics.example\`  

### Personal details extracted from OSINT
- Dog’s name: \`Bogi\`  
- Dog birth year: \`2020\`  
- User birthdate: \`1999-07-14\`  
- Hobbies: running, volleyball, Marvel movies  

These will be used to build password patterns later.

---

# 3. Username Format Enumeration

Most Windows environments use predictable username formats.  
From the email \`anna.kovacs@northwindlogistics.example\`, attackers derive:

- \`anna.kovacs\`  
- \`akovacs\`  
- \`kovacsa\`  
- \`anna.k\`  

Save this list on Kali for Scenario 2:

\`\`\`bash
echo -e "anna.kovacs\nakovacs\nkovacsa\nanna.k" > usernames.txt
\`\`\`

**Screenshot later:**  
- Show \`usernames.txt\` inside terminal.

---

# 4. Password Seed Collection (OSINT-Based)

Extracted details become password “building blocks”:

- \`Bogi\`  
- \`Bogi2020\`  
- \`Bogi1999\`  
- \`0714\`  
- \`Bogi!\`  
- \`Marvel1999!\`  
- \`Running99!\`  

These will generate a focused password list used in Scenario 2.

---

# 5. Identifying the External Server IP

The attacker scans the public-facing server (Windows Server hosted in GCP).

Recorded:
- Windows Server (RDP) public IP: \`34.123.45.67\` (example)

Add optional hostname mapping on Kali:

\`\`\`bash
echo "34.123.45.67   rdp.northwindlogistics.example" | sudo tee -a /etc/hosts
\`\`\`

**This IP belongs to the corporate server — not the employee workstation.**

We still need the **user’s external IP**, which cannot be found via OSINT alone.

---

# 6. Capturing the User Workstation External IP  
**(This will be used in Scenario 3, but happens BEFORE all scenarios)**

The attacker sends a harmless link to the employee — when clicked, the attacker’s server logs:

- the user's external IP  
- user agent  
- OS  
- browser  

This simulates real-world phishing infrastructure used in APT & red team operations.

## 6.1 Set up a simple IP-capture server on Kali

Create a folder:

\`\`\`bash
mkdir ip-capture
cd ip-capture
\`\`\`

Create \`server.py\`:

\`\`\`python
from http.server import BaseHTTPRequestHandler, HTTPServer
import logging

class RequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        logging.info(f"Victim IP: {self.client_address[0]}, User-Agent: {self.headers.get('User-Agent')}")
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b"<h1>Tracking Error</h1><p>Please try again later.</p>")

def run():
    logging.basicConfig(filename='ip_log.txt', level=logging.INFO)
    server_address = ('0.0.0.0', 8080)
    httpd = HTTPServer(server_address, RequestHandler)
    print("Listening on port 8080...")
    httpd.serve_forever()

if __name__ == "__main__":
    run()
\`\`\`

Run the server:

\`\`\`bash
python3 server.py
\`\`\`

---

# 6.2 User clicks → attacker obtains:

Example log entry in \`ip_log.txt\`:

```
Victim IP: 89.132.17.44, User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

This IP represents either:
- the user workstation’s external IP, or  
- the corporate outbound gateway IP  

Both are **useful reconnaissance intelligence**.

**Screenshot:**  
- Display \`ip_log.txt\` after a click occurs.

---

# 7. OSINT-Based Password List Generation

Create the final password list for Scenario 2:

\`\`\`bash
cat << 'EOF' > osint_passwords.txt
Bogi
Bogi1
Bogi01
Bogi99
Bogi1999
Bogi2020
Bogi2024
Bogi2024!
Bogi0714
Bogi0714!
Bogi!2024
BogiKovacs1
BogiKovacs!
Anna1999
Anna0714
Anna!1999
Running99!
Marvel1999!
Marvel!99
EOF
\`\`\`

---

# ✔ End of OSINT Pre-Stage

Proceed to **Scenario 1 — Basic Brute Force Attack** once all OSINT data is collected.
