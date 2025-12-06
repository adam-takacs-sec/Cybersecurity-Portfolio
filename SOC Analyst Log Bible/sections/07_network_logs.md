# **Section 7 — Network Logs**
_Comprehensive SOC, Threat Hunting & Detection Engineering Reference_

Network logs are one of the strongest sources for detecting lateral movement, C2 (command and control), data exfiltration, malware communication, VPN misuse, and early signs of compromise.

This section covers:
- Firewall logs (Palo Alto, Fortinet, Cisco ASA)
- VPN logs (AnyConnect, GlobalProtect)
- DNS logs (server-side & endpoint-side)
- Proxy / Web Gateway logs (Squid, BlueCoat)

---

# 🧭 **TABLE OF CONTENTS**
- [Overview of Network Logging](#overview-of-network-logging)
- [Firewall Logs](#firewall-logs)
  - [Palo Alto](#palo-alto)
  - [Fortinet](#fortinet)
  - [Cisco ASA](#cisco-asa)
  - [Firewall SOC Red Flags](#firewall-soc-red-flags)
- [VPN Logs](#vpn-logs)
  - [AnyConnect](#anyconnect)
  - [GlobalProtect](#globalprotect)
  - [VPN SOC Red Flags](#vpn-soc-red-flags)
- [DNS Logs](#dns-logs)
  - [DNS Server Logs](#dns-server-logs)
  - [Endpoint DNS Logs](#endpoint-dns-logs)
  - [DNS SOC Red Flags](#dns-soc-red-flags)
- [Proxy / Web Gateway Logs](#proxy--web-gateway-logs)
  - [Squid](#squid)
  - [BlueCoat](#bluecoat)
  - [Proxy SOC Red Flags](#proxy-soc-red-flags)
- [SIEM Detection Queries](#siem-detection-queries)

---

# ---
# ## **Overview of Network Logging**

Network logs give visibility into:
- inbound & outbound traffic  
- lateral movement  
- port scans  
- malware beaconing  
- command & control channels  
- data exfiltration  
- VPN login behavior  
- web activity  

They are often the **first indication** of a compromised host.

---

# ---
# ## **Firewall Logs**

Firewalls provide detailed telemetry about allowed/denied traffic, including:
- source IP  
- destination IP  
- source port  
- destination port  
- protocol  
- action (allow/deny)  
- bytes sent/received  
- application (for Next-Gen firewalls)  

---

# ###
# # **Palo Alto**
Palo Alto logs include:
- App-ID  
- Threat-ID  
- URL categories  
- Traffic direction  
- NATed and original IPs  

### **Example Log**
```
src=10.0.1.15 dst=185.199.220.55 app=https action=allow
sent=2048 received=5632 rule=Outbound-Internet
```

### **SOC Signals**
- Internal → external unusual IP  
- Application mismatch (e.g., cmd.exe generating HTTPS traffic)  
- High volume outbound data  

---

# ###
# # **Fortinet**
Fortigate logs contain fields like:
- srcip, dstip  
- service  
- appid  
- action  
- msg  

### **Example**
```
srcip=10.0.2.11 dstip=91.220.44.19 srv=HTTPS action=accept msg="HTTPS request"
```

### **Detection Notes**
- Rare outbound connections  
- Connections to blacklisted IPs  
- Newly observed URLs  

---

# ###
# # **Cisco ASA**

Common fields:
- Start/stop connection  
- Duration  
- Bytes transferred  
- Source → destination details  

### **Example**
```
%ASA-6-302013: Built outbound TCP connection 192.168.15.33 to 45.77.93.12/443
```

### **Watch For**
- Unexpected outbound 80/443  
- Reverse shells  
- Encrypted C2  

---

# ---
# ## **Firewall SOC Red Flags**

### **1️⃣ Internal → External Traffic to Rare Countries**
- Russia  
- China  
- Iran  
- North Korea  
- Unknown hosting providers  

### **2️⃣ High-frequency beaconing**
Every:
- 10 seconds  
- 30 seconds  
- 1 minute  

Typical for:
- Cobalt Strike  
- Metasploit  
- Generic malware C2  

### **3️⃣ Internal Host Scanning Internal Subnets**
Indicates lateral movement.

### **4️⃣ Large outbound uploads**
Potential exfiltration.

### **5️⃣ Traffic over unusual ports**
- 8080  
- 8443  
- 53  
- 12345  
- 3389 out of the network  

### **6️⃣ Malware patterns**
- Internal → random VPS IP  
- External → reverse shells inbound  

---

# ---
# ## **VPN Logs**

VPN logs are crucial for detecting:
- compromised accounts  
- impossible travel  
- new device profiles  
- password spraying  
- credential theft  
- lateral movement starting outside network  

---

# ###
# # **AnyConnect**

Example successful login:
```
User: john.doe
Assigned IP: 10.8.1.33
Tunnel Group: Employee-VPN
Group Policy: Default
```

Failed login:
```
Reason: Authentication failed
User: admin
Remote IP: 185.55.12.81
```

---

# ###
# # **GlobalProtect**

Example:
```
user=john.doe@company.com type=Gateway-Login
public-ip=185.32.44.12 location=Hungary
status=success ip=10.20.22.31
```

---

# ---
# ## **VPN SOC Red Flags**

### **1️⃣ Impossible Travel**
Login from:
- Hungary → 15 min later → Germany  
- US → 10 min later → China  

Impossible physically → indicates credential theft.

### **2️⃣ Multiple Failed Logins Followed by Success**
Classic brute force detection.

### **3️⃣ New Device or New OS**
Attackers often log in from systems never used before.

### **4️⃣ Login Outside Normal Working Hours**
E.g. 03:00 AM login for an office employee.

### **5️⃣ Excessive reconnections**
Malware or unstable malicious tools.

---

# ---
# ## **DNS Logs**

DNS logs are powerful for detecting:
- malware callbacks  
- DGA (Domain Generation Algorithms)  
- phishing domain access  
- DNS tunneling  

DNS logs come from:
- DNS server logs  
- Endpoint DNS (Sysmon Event ID 22)  
- Firewall DNS logging  

---

# ###
# # **DNS Server Logs**

Example:
```
client 10.0.3.44#55321: query: bad-domain.xyz A IN
```

### **SOC Signals**
- Unknown TLDs (.xyz, .top, .club, .click)  
- Random subdomain patterns  
- High-frequency queries  
- TXT queries (DNS exfiltration)  

---

# ###
# ## **Endpoint DNS Logs**  
(Using Sysmon Event ID 22)

Example:
```
Image: powershell.exe
QueryName: f9ajd9a3fja9fjf-dga-domain.xyz
```

### **Detection Notes**
- Processes making DNS requests:  
  - powershell.exe  
  - rundll32.exe  
  - regsvr32.exe  
  - mshta.exe  
  - suspicious executables  

- Beaconing patterns  
- Random-subdomain DNS queries  

---

# ---
# ## **DNS SOC Red Flags**

### **1️⃣ DGA Domains**
```
as93jf9asjf9asjf9sa9fjs.com
```

### **2️⃣ Frequent TXT Record Queries**
Used for command & control.

### **3️⃣ DNS Tunneling**
- extremely long TXT requests  
- thousands of requests per minute  

### **4️⃣ DNS resolution by unexpected processes**

---

# ---
# ## **Proxy / Web Gateway Logs**

Proxy logs provide HTTP/HTTPS visibility including:
- URL  
- User-Agent  
- categories  
- upload/download size  
- referrer  

Used for:
- malware delivery detection  
- phishing detection  
- data exfiltration detection  
- C2 over HTTP(S)  

---

# ###
# # **Squid**

Example:
```
10.0.2.15 TCP_MISS/200 1024 GET http://malicious.site/payload - HIER_DIRECT/91.210.12.44
```

### **Red Flags**
- Downloads from IP addresses instead of domains  
- Executable downloads (.exe, .js, .ps1, .hta)  
- TOR exit node connections  

---

# ###
# # **BlueCoat**

BlueCoat logs include:
- category  
- reputation  
- risk level  
- user ID  

### **Example**
```
result=blocked user=john.doe category=Malware URL=http://185.3.22.51/payload.ps1
```

### **SOC Value**
- Blocks malicious downloads  
- Detects phishing URLs  
- Identifies potential C2  

---

# ---
# ## **Proxy SOC Red Flags**

### **1️⃣ Suspicious File Downloads**
- `.exe`, `.dll`, `.ps1`, `.rar`, `.zip`  
- from unknown domains  

### **2️⃣ Cloud Storage Exfiltration**
- Dropbox  
- Google Drive  
- Mega  
- anonfiles  
- temp.sh  

Large outbound uploads → high alert.

### **3️⃣ TOR / VPN Services**
- nordvpn.com  
- protonvpn.com  
- tor nodess  

### **4️⃣ User-Agent Spoofing**
Attackers disguise malware as:
```
Mozilla/5.0
curl/7.55
python-requests
```

### **5️⃣ Connections Directly to IPs**
Malware often bypasses domain names.

---

# ---
# ## **SIEM Detection Queries**

### **KQL — DNS DGA Detection**
```kql
DNSEvents
| where strlen(QueryName) > 35
| where QueryName matches regex "[a-zA-Z0-9]{20,}"
```

### **KQL — Proxy Suspicious File Download**
```kql
ProxyLogs
| where Url endswith ".exe" or Url endswith ".ps1" or Url endswith ".hta"
```

### **KQL — VPN Impossible Travel**
```kql
VPNLogs
| summarize min(TimeGenerated), max(TimeGenerated) by User, SourceIP, Country
| where Country != prev(Country)
```

### **KQL — Firewall Outbound to Rare IPs**
```kql
Firewall
| where DestinationCountry in ("Russia","China","Iran","Unknown")
```

### **KQL — Beaconing Detection**
```kql
Firewall
| summarize count() by bin(TimeGenerated, 1m), DestinationIP
| where count_ > 10
```

---

# 🎯 **End of Section 7**
This section provides full SOC visibility into firewalls, VPN activity, DNS analysis, and proxy logs — essential for detecting C2, exfiltration, and lateral movement.

Next:  
➡️ **Section 8 — Web Server Logs**
