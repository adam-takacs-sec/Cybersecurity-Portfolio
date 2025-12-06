# **Section 9 — Cloud Logs**
_Comprehensive SOC, DFIR & Threat Hunting Reference_

Cloud environments introduce new identity-centric attack surfaces.  
Most compromises in Azure, AWS, and GCP begin with **misused credentials, privilege escalation, or abnormal API activity**, not malware.

Cloud logs provide the essential visibility to detect:
- compromised accounts  
- MFA bypass / token theft  
- privilege escalation  
- identity abuse  
- resource manipulation  
- data exfiltration  
- persistence mechanisms  

This section covers:

- Azure AD Sign-in Logs  
- AWS CloudTrail  
- GCP Audit Logs  

---

# 🧭 **TABLE OF CONTENTS**
- [Cloud Logging Overview](#-cloud-logging-overview)
- [Azure AD Logs](#-azure-ad-logs)
  - [Sign-in Logs](#-azure-ad-sign-in-logs)
  - [Audit Logs](#-azure-ad-audit-logs)
  - [Risky Sign-ins](#-azure-ad-risky-sign-ins)
  - [Azure SOC Red Flags](#-azure-soc-red-flags)
- [AWS CloudTrail](#-aws-cloudtrail)
  - [Important Event Types](#-important-event-types)
  - [Attack Patterns](#-aws-attack-patterns)
  - [AWS SOC Red Flags](#-aws-soc-red-flags)
- [GCP Audit Logs](#-gcp-audit-logs)
  - [Key Event Categories](#-key-event-categories)
  - [GCP SOC Red Flags](#-gcp-soc-red-flags)
- [SIEM Queries](#-siem-queries)

---

# ---
# ## **Cloud Logging Overview**

Cloud monitoring is based on:
- identity events  
- resource access  
- API calls  
- configuration changes  
- network activity  
- authentication results  

The defender’s mindset must shift from:
**“What is running on the machine?”**  
to  
**“Who did what, from where, using which identity, and why?”**

Identity = the new perimeter.

---

# ---
# ## **Azure AD Logs**

Azure AD logs are essential for:
- user authentication  
- app authentication  
- token usage  
- conditional access  
- MFA enforcement  
- anomalous login attempts  

---

# ###
# # **Azure AD Sign-in Logs**

Captured when:
- a user signs into Azure  
- a user signs into Office 365  
- a user signs into Azure portal  
- app/API auth occurs  

### **Key Fields**
| Field | Meaning |
|--------|---------|
| userPrincipalName | identity used |
| appDisplayName | application accessed |
| ipAddress | origin |
| location | geo source |
| deviceDetail | OS, browser |
| authenticationRequirement | MFA, password, token |
| status | success/failure |

### **Real Example**
```
User: admin@company.com
App: Azure Portal
Status: Success
IPAddress: 185.34.22.11
Location: Russia
Auth Requirement: MFA not satisfied
```

### **Red Flags**
- Successful login from new country  
- MFA not required due to Conditional Access misconfiguration  
- Risky login events pre-attack  

---

# ###
# # **Azure AD Audit Logs**

Logs all directory-related changes:
- user creation  
- group membership changes  
- admin role assignment  
- password resets  
- application consent  
- service principal creation  

### **Real Example**
```
Activity: Add member to role
Role: Global Administrator
Actor: helpdesk@company.com
Target User: attacker@evil.com
Status: Success
```

### **Red Flags**
- Admin roles assigned unexpectedly  
- Service principals created silently  
- Application consent granted without approval (OAuth attack)  

---

# ###
# # **Azure AD Risky Sign-ins**

Risk engine classifies:
- atypical travel  
- impossible travel  
- anomalous token usage  
- password spray attempts  
- suspicious browser fingerprints  

### **SOC Value**
Risky events usually indicate:
- compromised accounts  
- token theft  
- brute-force success  
- session hijacking  

---

# ---
# ## **Azure SOC Red Flags**
- Login from TOR exit nodes  
- MFA bypass  
- Disabled security defaults  
- Newly created global admin  
- OAuth consent phishing  
- Excessive failed logins  
- Legacy authentication (IMAP/POP) used  
- Impossible travel  
- Token replay  

---

# ---
# ## **AWS CloudTrail**

CloudTrail logs **every API call** made in AWS.

AWS compromises often involve:
- stolen access keys  
- misused IAM permissions  
- resource manipulation  
- privilege escalation  

---

# ###
# # **Important Event Types**

### **1️⃣ Authentication**
```
ConsoleLogin
AssumeRole
GetSessionToken
```

### **2️⃣ IAM Modifications**
```
CreateUser
AttachUserPolicy
PutRolePolicy
CreateAccessKey
```

### **3️⃣ EC2 Activity**
```
RunInstances
TerminateInstances
CreateSecurityGroup
AuthorizeSecurityGroupIngress
```

### **4️⃣ S3 Access**
```
GetObject
PutObject
ListBuckets
```

### **5️⃣ Lambda Abuse**
```
InvokeFunction
UpdateFunctionCode
```

---

# ###
# ## **AWS Attack Patterns**

### **1️⃣ Attacker steals IAM key → Enumerates IAM**
```
iam:ListUsers
iam:ListAccessKeys
```

### **2️⃣ Attacker escalates privileges**
```
iam:PutRolePolicy
iam:AttachUserPolicy   → AdministratorAccess
```

### **3️⃣ Attacker deploys backdoor access keys**
```
iam:CreateAccessKey
```

### **4️⃣ Attacker disables CloudTrail**
```
cloudtrail:StopLogging
```

### **5️⃣ Attacker launches crypto miners**
```
ec2:RunInstances
```

---

# ---
# ## **AWS SOC Red Flags**
- CloudTrail logging disabled  
- New access key created for dormant user  
- IAM policy modified to allow *\** actions  
- EC2 instances launched in unusual regions  
- S3 bucket publicly exposed  
- Large data transfer out of S3  
- AssumeRole used by unexpected users  
- Console login without MFA  

---

# ---
# ## **GCP Audit Logs**

GCP logs cover:
- Admin Activity  
- Data Access  
- System Events  
- Policy Deny events  

---

# ###
# ## **Key Event Categories**

### **1️⃣ Admin Activity**
- IAM role assignments  
- service account modifications  
- firewall rule changes  

### **2️⃣ Data Access**
- database reads  
- Secret Manager access  
- storage bucket access  

### **3️⃣ System Events**
- compute instance start/stop  
- VM creation  
- autoscaling  

### **4️⃣ Policy Deny**
- blocked API calls  
- denied access attempts  

---

# ---
# ## **GCP SOC Red Flags**

### **1️⃣ Service Account Abuse**
Attackers often:
- impersonate service accounts  
- escalate privileges  
- access sensitive resources  

### **2️⃣ IAM Misconfigurations**
- overly permissive roles  
- custom roles with wildcard access  

### **3️⃣ Suspicious VM Activity**
- crypto mining  
- unusual regions  
- large data movement  

### **4️⃣ Storage Misuse**
- public bucket exposure  
- exfiltration via bucket downloads  

### **5️⃣ API Scanning**
Automated enumeration:
```
compute.instances.list
iam.serviceAccounts.list
storage.buckets.list
```

---

# ---
# ## **SIEM Queries**

### **KQL — Detect Impossible Travel (Azure)**
```kql
AzureSignIn
| summarize min(TimeGenerated), max(TimeGenerated) by User, Country
| where Country != prev(Country)
```

### **KQL — New Global Administrator**
```kql
AzureAudit
| where Activity matches "Add member to role"
| where Role == "Global Administrator"
```

### **KQL — AWS New Access Key**
```kql
CloudTrail
| where eventName == "CreateAccessKey"
```

### **KQL — AWS Privilege Escalation**
```kql
CloudTrail
| where eventName in ("AttachUserPolicy","PutRolePolicy")
| where requestParameters contains "AdministratorAccess"
```

### **KQL — GCP Suspicious Service Account Use**
```kql
GCPAudit
| where ProtoPayload.ServiceName contains "iam"
| where ProtoPayload.MethodName contains "signBlob"
```

---

# 🎯 **End of Section 9**
This section provides full coverage of Azure, AWS, and GCP logs to detect identity-based attacks, privilege escalation, data exfiltration, and cloud account compromise.

---

### ➡️ Next: [Section 10 — EDR Telemetry](./10_edr_telemetry.md)

