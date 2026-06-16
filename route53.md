# 🌐 AWS Route 53 — DNS Service | Hinglish Notes
> **Source:** Chai aur Code — AWS Zero to Hero Course  
> **Topic:** Route 53 — Scalable DNS & Traffic Routing  
> **Coverage:** DNS Basics → Domain Registration → Hosted Zones → DNS Records → Routing Policies → Health Checks → Billing

---

## 📌 Table of Contents

1. [Route 53 kya hai?](#1-route-53-kya-hai)
2. [DNS kaise kaam karta hai?](#2-dns-kaise-kaam-karta-hai)
3. [Domain Registration](#3-domain-registration)
4. [Hosted Zone](#4-hosted-zone)
5. [DNS Record Types](#5-dns-record-types)
6. [Routing Policies](#6-routing-policies)
7. [Health Checks](#7-health-checks)
8. [Latency-Based Routing — Practical Setup](#8-latency-based-routing--practical-setup)
9. [Failover Demo](#9-failover-demo)
10. [Route 53 Use Cases](#10-route-53-use-cases)
11. [Billing](#11-billing)
12. [Cleanup](#12-cleanup)
13. [Quick Revision](#13-quick-revision)

---

## 1. Route 53 kya hai?

### 📖 Definition

> **Route 53 = AWS ka Scalable DNS (Domain Name System) Service**

Simple bolo toh:
- Tumhari website abhi **IP address** se access hoti hai (e.g., `54.123.45.67`)
- Route 53 se tum **custom domain** use kar sakte ho (e.g., `mywebsite.com`)
- Route 53 domain name ko IP mein **translate** karta hai aur traffic sahi server pe bhejta hai

---

### 🔢 Route 53 naam kyun?

```
DNS ka default port = 53
Route + 53 = Route 53 🎯
```

---

### 🌟 Route 53 kya-kya karta hai?

```
1. Domain Registration  → Domain khareed/register karo
2. DNS Management       → Domain → IP mapping
3. Traffic Routing      → Kahan se serve ho — decide karo
4. Health Checks        → Server down? Auto-reroute!
```

---

## 2. DNS kaise kaam karta hai?

### 🔄 Flow Diagram

```
User types: www.mywebsite.com
              ↓
        Browser request
              ↓
       DNS Server (Route 53)
              ↓
   Domain → IP translation
   mywebsite.com → 54.123.45.67
              ↓
   EC2 Instance / Server
              ↓
       Website load hoti hai ✅
```

> 💡 **Analogy:** DNS ek **Phone Book** ki tarah hai — naam diya, number (IP) mila!

---

### 🏗️ Complete Architecture

```
Browser → Route 53 (DNS) → EC2/S3/Any Server

Without Route 53:  54.123.45.67   (IP yaad rakhna padta hai)
With Route 53:     mywebsite.com  (naam se access karo!) ✅
```

---

## 3. Domain Registration

### 🛒 Domain kahan se khareedein?

#### Option 1: AWS Route 53 se (directly)

```
AWS Console → Route 53 → "Register Domain"
→ Domain naam search karo (e.g., mywebsite.com)
→ Available hai? → Checkout → Pay → Done!

Pricing: $14/year se $303/year (extension ke hisaab se)
  .com → ~$14/year
  .net → ~$14/year
  .io  → ~$40/year
```

#### Option 2: Third-party se (Hostinger, GoDaddy, etc.)

```
Hostinger.in → Domain search → Buy
Pricing: ₹99/year se (bahut sasta!)

Fayda: Bahut saste mein milta hai
Kaam: Baad mein nameservers Route 53 ke point karne padte hain
```

> 💡 **Recommendation:** Pehli baar ke liye Hostinger ya GoDaddy se cheap domain lo — concept samajhne ke liye!

---

### ⚠️ Important Note — Route 53 is PAID!

```
❌ Route 53 mein FREE TIER nahi hai!

Billable components:
  - Hosted Zone: ~$0.50/month per zone
  - DNS Queries: per million queries charge
  - Health Checks: per health check/month
  - Domain Registration: yearly fee
```

> Sirf concept samajhna hai toh practical skip kar sakte ho — theory se bhi samajh aa jaayegi!

---

## 4. Hosted Zone

### 📖 Kya hota hai?

> **Hosted Zone = Route 53 mein ek container jo tumhare domain ke DNS records rakhta hai**

```
Hosted Zone: mywebsite.com
  ├── A Record    → mywebsite.com → 54.123.45.67
  ├── CNAME Record → www → mywebsite.com
  └── NS Record   → Name Servers (auto-generated)
```

---

### 🛠️ Hosted Zone create karna

#### Case 1: Domain AWS se liya → Auto create!
```
Domain register karte hi Hosted Zone automatically ban jaata hai ✅
```

#### Case 2: Domain bahar se liya (Hostinger, etc.)

```
Route 53 → "Hosted Zones" → "Create Hosted Zone"
→ Domain Name: mywebsite.com (ya jo bhi tumhara domain hai)
→ Type: Public Hosted Zone
→ Create

✅ Hosted Zone ban gayi!
```

---

### 🔗 Nameservers Hostinger pe point karna

Hosted Zone banate hi **4 Nameservers** milte hain:

```
ns-123.awsdns-12.com
ns-456.awsdns-34.net
ns-789.awsdns-56.org
ns-012.awsdns-78.co.uk
```

Inhe Hostinger mein update karo:

```
Hostinger Dashboard → Domain → DNS → Change Nameservers
→ Ye 4 nameservers paste karo → Save

⏰ Changes propagate hone mein 24-48 hours lag sakte hain!
```

---

### 🔄 Flow after Nameserver Update

```
User: mywebsite.com type karta hai
         ↓
Hostinger: "Ye domain Route 53 ke paas ja!"
         ↓
Route 53: "Haan, iska IP 54.123.45.67 hai"
         ↓
Browser: EC2 Instance pe request bhejta hai ✅
```

---

## 5. DNS Record Types

### 📋 Sabse Important Record Types

| Record Type | Kya karta hai | Example |
|---|---|---|
| **A** | Domain → IPv4 IP | `mywebsite.com → 54.123.45.67` |
| **AAAA** | Domain → IPv6 IP | `mywebsite.com → 2001:db8::1` |
| **CNAME** | Domain → Domain | `www.mywebsite.com → mywebsite.com` |
| **NS** | Name Servers define karta hai | `mywebsite.com → ns-123.awsdns.com` |
| **MX** | Mail servers ke liye | `mail.mywebsite.com → mailserver` |
| **TXT** | Text info (verification ke liye) | Domain ownership verify |
| **SRV** | Specific services ke liye | VoIP, gaming servers |

---

### 🔍 A Record vs CNAME — Difference

```
A Record:
  mywebsite.com → 54.123.45.67 (IP address)
  Direct IP mapping

CNAME Record:
  www.mywebsite.com → mywebsite.com (domain to domain)
  google.com → www.google.com → [IP]

Use CNAME when:
  - www.mysite.com → mysite.com redirect
  - blog.mysite.com → mysite.com
  - mail.mysite.com → mysite.com
```

---

### 🔖 Alias Record (AWS specific)

> **Alias = AWS specific feature** — IP ki jagah directly AWS resource point karo

```
Normal A Record: Domain → IP address
Alias Record:    Domain → AWS Resource (S3, ELB, CloudFront, etc.)

Example:
  mywebsite.com → ALIAS → S3 bucket URL
  mywebsite.com → ALIAS → Load Balancer DNS
```

> 💡 **Fayda:** S3 website ya Load Balancer ka IP change ho sakta hai — Alias automatically update hota hai!

---

## 6. Routing Policies

### 📖 Kya hoti hain?

> **Routing Policies = Rules jo decide karte hain ki DNS query ka response kaise aur kahan bheja jaaye**

---

### 📊 Routing Policies — Complete Table

| Policy | Kab use karo | How it works |
|---|---|---|
| **Simple** | Single server, basic setup | Ek hi IP return karta hai |
| **Weighted** | A/B testing, gradual traffic shift | 70% → Server A, 30% → Server B |
| **Latency-based** | Global users, low latency chahiye | User ke nearest server pe route karo |
| **Failover** | High availability, disaster recovery | Primary down? → Secondary pe jaao |
| **Geolocation** | Content localization | India se aaya? → India server |
| **Geoproximity** | Fine-grained geo control | Specific coordinates se route karo |
| **Multi-value** | Multiple IPs, basic load balance | Multiple IPs return karta hai |

---

### 1️⃣ Simple Routing

```
User → DNS → mywebsite.com → 54.123.45.67 → EC2 Instance

Use case: Testing, simple single-server setup
```

---

### 2️⃣ Weighted Routing

```
Record 1: mywebsite.com → Server A (Weight: 70)
Record 2: mywebsite.com → Server B (Weight: 30)

Result:
  70% traffic → Server A
  30% traffic → Server B

Use case:
  - A/B testing (new version test karna)
  - Gradual traffic migration
  - Blue-Green deployment
```

---

### 3️⃣ Latency-Based Routing

```
Setup:
  Record 1: mywebsite.com → Stockholm EC2 (Region: eu-north-1)
  Record 2: mywebsite.com → Mumbai EC2   (Region: ap-south-1)

Result:
  European user  → Stockholm server serve karega (low latency)
  Indian user    → Mumbai server serve karega (low latency)

Use case: Global websites jahan different regions mein servers hain
```

---

### 4️⃣ Failover Routing

```
Primary:   mywebsite.com → Server A (Main server)
Secondary: mywebsite.com → Server B (Backup)

Normal:    All traffic → Server A
If A down: All traffic → Server B (automatic!)

Use case: High availability, zero downtime goal
```

---

### 5️⃣ Geolocation Routing

```
India users     → India server
US users        → US server
Europe users    → Europe server
Others          → Default server

Use case:
  - Hindi content → Indian users
  - GDPR compliance → European users
  - Language-specific websites
```

---

## 7. Health Checks

### 📖 Kya hota hai?

> **Health Check = Route 53 regularly tumhare server ko ping karta hai — healthy hai ya nahi check karta hai**

```
Route 53 → Every 30 seconds → Server ko ping karta hai
              ↓
  Server responds?  → ✅ Healthy
  Server down?      → ❌ Unhealthy → Traffic reroute!
```

---

### 🛠️ Health Check Create karna

```
Route 53 → "Health Checks" → "Create Health Check"

Settings:
  Name: eu-health-check
  What to monitor: Endpoint
  Specify endpoint by: IP Address
  Protocol: HTTP
  IP Address: <EC2 public IP>
  Port: 80
  Path: / (root path check karega)
  
Advanced:
  Request interval: 30 seconds (ya 10 seconds — faster check)
  Failure threshold: 3 (3 baar fail ho tab unhealthy declare karo)

→ Create ✅
```

---

### 🔗 Health Check ko Record se Link karna

```
Hosted Zone → Record edit karo
→ Health Check ID: apna health check select karo
→ Save

Ab agar server down hua:
  Health check → Unhealthy detect karega
  Route 53 → Us record se traffic hataa dega
  Traffic → Healthy server pe jaayega ✅
```

---

### 📊 Health Check Status dekhna

```
Route 53 → Health Checks → Status column:

  ✅ Healthy    → Server sahi chal raha hai
  ❌ Unhealthy  → Server down! Traffic reroute ho raha hai
```

---

## 8. Latency-Based Routing — Practical Setup

### 🎯 Goal

```
2 EC2 instances:
  EU Instance   → Stockholm (Europe users ke liye)
  Asia Instance → Mumbai    (Indian/Asian users ke liye)

DNS aise route kare ki har user apne closest server se serve ho!
```

---

### 📋 Step-by-Step Setup

#### Step 1: 2 EC2 Instances banao

```
Instance 1:
  Region: eu-north-1 (Stockholm)
  Name: my-web-server-EU
  User Data Script:
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Apache Web Server from EUROPE</h1>" > /var/www/html/index.html

Instance 2:
  Region: ap-south-1 (Mumbai)
  Name: my-web-server-Asia
  User Data Script:
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Apache Web Server from MUMBAI</h1>" > /var/www/html/index.html
```

#### Step 2: Dono IPs note karo

```
EU Instance Public IP:   54.XXX.XX.XX
Asia Instance Public IP: 13.XXX.XX.XX
```

#### Step 3: Hosted Zone mein Records banao

**Record 1 — Europe:**
```
Route 53 → Hosted Zone → Create Record

Record Name: www (ya blank for root)
Record Type: A
Value: 54.XXX.XX.XX (EU instance IP)
Routing Policy: Latency
Region: eu-north-1
Record ID: EU-record
→ Create ✅
```

**Record 2 — Mumbai:**
```
Route 53 → Hosted Zone → Create Record (same naam!)

Record Name: www
Record Type: A
Value: 13.XXX.XX.XX (Mumbai instance IP)
Routing Policy: Latency
Region: ap-south-1
Record ID: Asia-record
→ Create ✅
```

#### Step 4: Test karo

```
India se access karo → "Apache Web Server from MUMBAI" dikhega ✅
Europe se access karo → "Apache Web Server from EUROPE" dikhega ✅
```

---

## 9. Failover Demo

### 🔴 Scenario: EU Server Down ho jaata hai

```
Before:
  Europe user → EU Server (low latency) ✅

EU Server terminate karo → Health Check → Unhealthy!

After failover:
  Europe user → Mumbai Server (fallback) ✅
  (Thoda slow, lekin website UP hai!)
```

### 👁️ Kaise verify karein?

```
1. EU EC2 instance terminate karo
2. Route 53 → Health Checks → EU health check → Status: Unhealthy
3. Browser → mywebsite.com → Refresh karo
4. Ab "from MUMBAI" dikhega — even Europe se access karoge!
```

> 💡 **Ye hai Failover ka magic** — Zero downtime, automatic rerouting!

---

## 10. Route 53 Use Cases

### 🎯 Common Use Cases

| Use Case | Solution |
|---|---|
| **Website ka custom domain** | A Record → EC2 IP |
| **S3 static website pe domain** | Alias Record → S3 endpoint |
| **Load balancing (manual)** | Weighted routing (50-50 ya custom) |
| **Global low-latency website** | Latency-based routing |
| **Disaster recovery** | Failover routing + Health Checks |
| **Region-wise content** | Geolocation routing |
| **Blue-Green deployment** | Weighted (start 90-10, gradually shift) |
| **Mail server setup** | MX Records |
| **Domain verification (SSL, etc.)** | TXT Records |

---

### 🌍 Multi-Region Deployment Architecture

```
                     Route 53
                    /    |    \
                   /     |     \
               EU      India    US
              Server   Server  Server
            (Stockholm)(Mumbai)(Virginia)

Route 53 → User ke nearest server pe bhejta hai
Health Check → Koi down hua → Traffic redistribute karo
```

---

## 11. Billing

### 💰 Route 53 Pricing (Approximate)

| Component | Cost |
|---|---|
| **Hosted Zone** | ~$0.50/month per zone |
| **DNS Queries** | $0.40 per million queries (first 1B) |
| **Health Checks** | $0.50/month per endpoint |
| **Domain Registration** | $9–$303/year (extension ke hisaab se) |
| **Traffic Flow Policies** | $50/month per policy record |

> ⚠️ **Free Tier mein Route 53 nahi aata!**  
> Pehle din se hi charge hona shuru ho jaata hai.

---

### 💡 Cost Minimize karne ke Tips

```
✅ Testing ke baad resources delete karo
✅ Zyada Health Checks mat banao
✅ Simple routing use karo jab advanced na chahiye
✅ Domain third-party se lo (Hostinger, etc.) — sasta padta hai
✅ Unused Hosted Zones delete karo
```

---

## 12. Cleanup

### 🧹 Cleanup Order (Important!)

```
Step 1: Health Checks delete karo
  Route 53 → Health Checks → Select all → Delete

Step 2: DNS Records delete karo
  Route 53 → Hosted Zones → Zone open karo
  → Custom records select karo (NS aur SOA mat karo!)
  → Delete Records

Step 3: Hosted Zone delete karo
  Route 53 → Hosted Zones → Delete
  (Pehle sab records delete hone chahiye)

Step 4: EC2 Instances terminate karo
  EC2 → Global View → All running instances
  → Terminate all

Step 5: Nameservers revert karo (agar Hostinger use kiya)
  Hostinger → Domain → DNS → Default nameservers pe wapas jaao
```

> 💡 **EC2 Global View trick:** Ek hi jagah se saare regions ke instances dekho — koi miss na ho!

---

## 13. Quick Revision

### 🔄 Route 53 Complete Flow

```
1. Domain khareedna:
   AWS Route 53 se → Direct
   Ya Hostinger se → Sasta option

2. Hosted Zone banana:
   AWS domain liya → Auto create
   External domain → Manual create → Nameservers copy karo

3. Nameservers update karna:
   (Sirf external domain ke liye)
   Hostinger → DNS → AWS nameservers paste karo
   Wait: 24-48 hours propagation

4. DNS Records banana:
   A Record → EC2 IP point karo
   CNAME → www ko root pe point karo
   Alias → S3 ya ALB pe point karo

5. Routing Policy choose karo:
   Single server → Simple
   Global users → Latency-based
   Backup chahiye → Failover
   A/B test → Weighted
   Region-wise → Geolocation

6. Health Checks:
   Har server ke liye ek health check
   Records se link karo
   Server down? → Auto reroute!

7. Test karo → Cleanup karo!
```

---

### 📝 Key Terms Glossary

| Term | Matlab |
|---|---|
| **Route 53** | AWS ka DNS + Domain registration service |
| **DNS** | Domain Name System — naam ko IP mein convert karta hai |
| **Domain** | Website ka naam (e.g., mywebsite.com) |
| **Hosted Zone** | Domain ke DNS records ka container in Route 53 |
| **DNS Record** | Domain → IP ya Domain → Domain mapping |
| **A Record** | Domain → IPv4 address |
| **AAAA Record** | Domain → IPv6 address |
| **CNAME** | Domain → Another domain |
| **NS Record** | Name Servers — DNS handle karta hai |
| **MX Record** | Mail server records |
| **TXT Record** | Text info (verification, SPF, etc.) |
| **Alias Record** | AWS resource ko directly point karna (S3, ALB) |
| **Nameserver** | DNS requests handle karne wala server |
| **Simple Routing** | Basic — ek IP return karo |
| **Weighted Routing** | Traffic percentage se divide karo |
| **Latency Routing** | Nearest server pe route karo |
| **Failover Routing** | Primary down? Secondary pe jaao |
| **Geolocation Routing** | User ke location se decide karo |
| **Health Check** | Server alive hai ya nahi — monitor karta hai |
| **Failover** | Automatic reroute when server fails |
| **Propagation** | DNS changes duniya mein spread hone mein time |
| **TTL** | Time to Live — DNS cache kitni der tak valid hai |

---

### ⚡ Common Mistakes to Avoid

```
❌ Hosted Zone delete karne se pehle records delete nahi kiye
❌ External domain ke nameservers update karna bhool gaye
❌ DNS propagation (24-48 hrs) ka wait nahi kiya
❌ Multiple regions mein EC2 banaye aur cleanup bhool gaye → Bill aaya!
❌ Health Check ko DNS Record se link nahi kiya
❌ Route 53 free hai socha — charges aa gaye!
❌ Same naam ke dono records banane chahiye the (Latency routing mein) — bhool gaye
```

---

### 🔑 Latency Routing Setup Checklist

```
□ EC2 Instance 1: Region 1 (e.g., Stockholm) — website running
□ EC2 Instance 2: Region 2 (e.g., Mumbai)   — website running
□ Hosted Zone created
□ Record 1: Latency → EU IP → Region: eu-north-1 → Record ID: EU
□ Record 2: Latency → Mumbai IP → Region: ap-south-1 → Record ID: Asia
□ Health Check 1: EU instance IP
□ Health Check 2: Mumbai instance IP
□ Both health checks linked to respective records
□ Test: India se access → Mumbai dikhna chahiye
□ Test: Europe se access → Stockholm dikhna chahiye
□ CLEANUP: Health checks → Records → Hosted Zone → EC2 instances
```

---

> 🚀 **Next Topics:** VPC (Virtual Private Cloud), Load Balancer, Auto Scaling, CloudFront (CDN)