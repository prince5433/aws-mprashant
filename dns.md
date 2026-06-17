# 🌐 DNS — Deep Dive with Practical | Hinglish Notes
> **Source:** Chai aur Code — AWS Zero to Hero Course  
> **Topic:** DNS Server Setup from Scratch — Khud ka DNS Server banana!  
> **Coverage:** DNS Theory → Web Server Setup → BIND9 DNS Server → Zone Files → DNS Records → Browser Testing

---

## 📌 Table of Contents

1. [DNS kya hota hai?](#1-dns-kya-hota-hai)
2. [Practical ka Goal](#2-practical-ka-goal)
3. [Setup Overview](#3-setup-overview)
4. [Step 1: Web Server Setup (Apache/httpd)](#4-step-1-web-server-setup-apachehttpd)
5. [Step 2: DNS Server Setup (BIND9)](#5-step-2-dns-server-setup-bind9)
6. [Step 3: DNS Configuration Files Edit karna](#6-step-3-dns-configuration-files-edit-karna)
7. [Step 4: Zone File banana](#7-step-4-zone-file-banana)
8. [Step 5: Service Restart aur Verify](#8-step-5-service-restart-aur-verify)
9. [Step 6: Browser se Access karna](#9-step-6-browser-se-access-karna)
10. [DNS Record Types](#10-dns-record-types)
11. [Zone File Types](#11-zone-file-types)
12. [Quick Revision](#12-quick-revision)

---

## 1. DNS kya hota hai?

### 📖 Definition

> **DNS = Domain Name System**  
> *"Human-friendly domain names ko machine-readable IP addresses mein translate karta hai"*

---

### 🧠 DNS kaise kaam karta hai?

```
User ne browser mein type kiya: www.google.com

Browser:
  "Mujhe pata nahi kaunse server pe jaana hai"
        ↓
  DNS Server se puchha: "www.google.com ka IP kya hai?"
        ↓
  DNS Server ne check kiya apni mapping:
    google.com → 142.250.77.46
        ↓
  Browser ne us IP pe request bheji
        ↓
  Google ka page load hua! ✅
```

---

### 📱 Real-Life Analogy

```
DNS = Phone Book / Contacts App

Tum "Rahul" naam se dhundhte ho → Number milta hai → Call karte ho
Tum google.com type karte ho → IP milti hai → Website khulti hai

Without DNS:
  142.250.77.46 yaad rakhna padta (impossible!)

With DNS:
  www.google.com likhdo → DNS baki sambhal leta hai ✅
```

---

### 🔢 DNS ka Default Port

```
DNS Port = 53
(Isliye AWS ki DNS service ka naam "Route 53" hai!)
```

---

## 2. Practical ka Goal

### 🎯 Hum kya banayenge?

```
Final Result:
  Browser mein type karo: www.myweb.com
  → Tumhari khud ki website open ho! ✅

Setup:
  ┌──────────┐     ┌──────────────┐     ┌────────────────┐
  │  Browser │────▶│  DNS Server  │────▶│   Web Server   │
  │          │     │  (BIND9)     │     │  (Apache/httpd)│
  │www.myweb │     │              │     │                │
  │  .com    │     │ myweb.com    │     │ 10.21.55.1     │
  └──────────┘     │→10.21.55.1   │     │                │
                   └──────────────┘     └────────────────┘
```

> ⚠️ **Note:** Ye website internet pe accessible nahi hogi — same network mein hi kaam karegi. Sirf DNS concept samajhne ke liye!

---

### 🖥️ Requirements

```
✅ Linux Virtual Machine (CentOS/RHEL/Rocky Linux recommended)
✅ Root user ya sudo privileges
✅ Basic Linux terminal knowledge
✅ Internet connection (packages download ke liye)
```

---

## 3. Setup Overview

### 📋 Step-by-Step Plan

```
Step 1: Web Server (Apache/httpd) install + setup karo
        → Website accessible ho jaye IP se

Step 2: DNS Server (BIND9) install karo

Step 3: Firewall mein DNS port allow karo

Step 4: BIND Configuration edit karo
        → Named.conf → Zone block add karo

Step 5: Zone File banao
        → Domain → IP mapping define karo

Step 6: Service restart karo

Step 7: resolv.conf edit karo
        → System ko batao apna DNS server use karo

Step 8: Browser se www.myweb.com access karo ✅
```

---

## 4. Step 1: Web Server Setup (Apache/httpd)

### 📦 httpd Install karna

```bash
# httpd package install karo (Apache Web Server)
yum install -y httpd

# Service start karo
systemctl start httpd

# Boot pe auto-start enable karo
systemctl enable httpd

# Status check karo
systemctl status httpd
```

**Output check karo:**
```
● httpd.service - The Apache HTTP Server
   Active: active (running) ✅
```

---

### 🌐 Website Content daalna

```bash
# Apache ka default web root
/var/www/html/

# Custom HTML page banao
vi /var/www/html/index.html
```

**Simple HTML example:**
```html
<!DOCTYPE html>
<html>
<head><title>My Website</title></head>
<body>
  <h1>Welcome to My Custom Website!</h1>
  <p>DNS is working! 🎉</p>
</body>
</html>
```

```bash
# Changes ke baad restart karo
systemctl restart httpd
```

---

### 🔥 Firewall mein HTTP allow karna

```bash
# HTTP service allow karo
firewall-cmd --add-service=http --permanent

# Firewall reload karo
firewall-cmd --reload

# Verify karo
firewall-cmd --list-services
```

---

### ✅ Website Test karna

```bash
# VM ke andar se test karo
curl http://10.21.55.1
# Ya browser mein: http://10.21.55.1
```

**Expected:** Tumhara HTML page dikhna chahiye!

---

## 5. Step 2: DNS Server Setup (BIND9)

### 📦 BIND9 Install karna

```bash
# bind package install karo
yum install -y bind bind-utils

# Verify installation
rpm -qa | grep bind
```

---

### ▶️ Named Service Start karna

```bash
# Service name hai "named" (not "bind")
systemctl start named

# Auto-start enable karo
systemctl enable named

# Status check karo
systemctl status named
```

---

### 🔥 Firewall mein DNS allow karna

```bash
# DNS service allow karo (port 53)
firewall-cmd --add-service=dns --permanent

# Firewall reload
firewall-cmd --reload
```

---

## 6. Step 3: DNS Configuration Files Edit karna

### 📁 Important File Locations

```
/etc/named.conf          → Main BIND configuration file
/var/named/              → Zone files ki directory
/etc/resolv.conf         → System ka DNS server setting
```

---

### 📝 /etc/named.conf Edit karna

```bash
# Pehle backup lo! (Best practice)
cp /etc/named.conf /etc/named.conf.bak

# File edit karo
vi /etc/named.conf
```

#### Original content (kuch aisa dikhega):

```
options {
    listen-on port 53 { 127.0.0.1; };
    ...
};
```

#### Changes karo:

**Change 1: Apni server IP add karo listen-on mein**

```
options {
    listen-on port 53 { 127.0.0.1; 10.21.55.1; };
    //                               ^^^^^^^^^^^
    //                               Apni server IP add karo!
    ...
};
```

**Change 2: Zone Block add karo (apna domain define karo)**

```
// Ye block "include" line se PEHLE add karo:

zone "myweb.com" IN {
    type master;
    file "myweb.com.fzone";
    allow-query { any; };
};
```

> 💡 **Explanation:**
> - `zone "myweb.com"` → Tumhara domain naam
> - `type master` → Ye primary DNS server hai
> - `file "myweb.com.fzone"` → Zone file ka naam (baad mein banayenge)
> - `allow-query { any; }` → Koi bhi query kar sake

---

### ✅ Configuration Verify karna

```bash
# Config file check karo (koi error nahi hona chahiye)
named-checkconf

# Agar koi output nahi aaya = ✅ File correct hai
# Error aaya = ❌ File mein kuch galat hai, fix karo
```

---

## 7. Step 4: Zone File banana

### 📖 Zone File kya hoti hai?

> **Zone File = Woh file jisme actual DNS records hain — Domain → IP mapping**

```
Zone File mein hoga:
  www.myweb.com → 10.21.55.1

Browser → DNS Server → Zone File check karega → IP return karega
```

---

### 📁 Zone File Create karna

```bash
# /var/named/ directory mein jaao
cd /var/named/

# Zone file banao (naam named.conf mein diya tha)
touch myweb.com.fzone

# Edit karo
vi myweb.com.fzone
```

---

### 📝 Zone File Content

```
$TTL 2D
@       IN      SOA     ns1.myweb.com.    admin.myweb.com. (
                        800     ; Serial Number
                        3600    ; Refresh
                        1800    ; Retry
                        604800  ; Expire
                        86400 ) ; Minimum TTL

@       IN      NS      ns1.myweb.com.
ns1     IN      A       10.21.55.1
www     IN      A       10.21.55.1
```

---

### 🔍 Zone File Explained — Line by Line

#### Line 1: `$TTL 2D`
```
TTL = Time To Live
2D  = 2 Days

Matlab: DNS cache 2 din tak valid rahega
        2 din baad DNS server se fresh info fetch hogi
```

#### SOA Record Block:
```
@    IN    SOA    ns1.myweb.com.    admin.myweb.com. (...)

@           → Ye domain itself ko represent karta hai (myweb.com)
IN          → Internet class
SOA         → Start of Authority (zone ka primary record)
ns1.myweb.com.  → Primary name server (DNS server ka naam)
admin.myweb.com. → Admin email (admin@myweb.com as DNS format)
```

#### SOA Numbers:
```
800      → Serial Number (file edit karne pe increment karo!)
3600     → Refresh: Secondary DNS kitni baar primary se sync kare (seconds)
1800     → Retry: Sync fail hua? Kitni der baad retry kare (seconds)
604800   → Expire: Secondary NS kab tak purana data use kare (7 days)
86400    → Minimum TTL: Negative caching duration (1 day)
```

#### NS Record:
```
@    IN    NS    ns1.myweb.com.

NS = Name Server record
Matlab: myweb.com ka name server = ns1.myweb.com
```

#### A Records:
```
ns1    IN    A    10.21.55.1
www    IN    A    10.21.55.1

A Record = Hostname → IPv4 IP mapping

ns1.myweb.com → 10.21.55.1  (DNS server ka IP)
www.myweb.com → 10.21.55.1  (Website ka IP)
```

> ⚠️ **Important:** Lines ke end mein dots (`.`) ka dhyan rakho! `ns1.myweb.com.` (dot required at end)

---

### ✅ Zone File Verify karna

```bash
# Zone file check karo
named-checkzone myweb.com /var/named/myweb.com.fzone

# Expected output:
zone myweb.com/IN: loaded serial 800
OK ✅

# Agar error aaya → File mein galti hai, fix karo
```

---

### 📁 File Location Note

```
/etc/named.conf mein likha:
  file "myweb.com.fzone";

BIND automatically /var/named/ mein dhundhta hai!

Agar file kahin aur hai toh full path do:
  file "/custom/path/myweb.com.fzone";
```

---

## 8. Step 5: Service Restart aur Verify

### 🔄 Named Service Restart karna

```bash
# DNS service restart karo (changes apply honge)
systemctl restart named

# Status verify karo
systemctl status named
```

---

### 🔍 nslookup se Verify karna

```bash
# nslookup tool use karo DNS test ke liye

# Normal internet DNS check (ye tumhara DNS use nahi karega)
nslookup www.google.com

# Tumhara DNS server specify karke check karo
nslookup www.myweb.com 10.21.55.1
#                      ^^^^^^^^^^
#                      Tumhara DNS server IP

# Expected output:
Server:    10.21.55.1
Address:   10.21.55.1#53

Name:   www.myweb.com
Address: 10.21.55.1   ✅
```

---

### 📝 resolv.conf Edit karna (System DNS change)

```bash
# System ka default DNS server change karo
vi /etc/resolv.conf

# Ye line change karo ya add karo:
nameserver 10.21.55.1   ← Tumhara DNS server

# Save karo
```

```bash
# Ab bina IP specify kiye test karo
nslookup www.myweb.com

# System automatically tumhara DNS use karega
# Expected:
Address: 10.21.55.1 ✅
```

---

## 9. Step 6: Browser se Access karna

### 🌐 Method 1: VM ke Andar Browser

```
Agar tumhare VM mein browser hai:

URL: www.myweb.com
→ Website khulegi! ✅
```

---

### 🌐 Method 2: Local System Browser DNS Change karna

#### Windows:
```
Control Panel → Network → IPv4 Properties
→ "Use the following DNS server addresses"
→ Preferred DNS server: 10.21.55.1
→ OK

Browser mein: www.myweb.com ✅
```

#### Mac:
```
System Settings → Network → WiFi → Details
→ DNS tab → Plus (+) click karo
→ DNS Server: 10.21.55.1 add karo
→ OK

Browser mein: www.myweb.com ✅
```

#### Chrome Browser DNS change (temporary):
```
Chrome Settings → Privacy and Security
→ Security → Use secure DNS
→ "With Custom" → 10.21.55.1 enter karo

Browser mein: www.myweb.com ✅
```

---

### ⚠️ Important Note — Sirf tumhari website accessible!

```
Jab tum apna DNS server point karte ho:

www.myweb.com  → ✅ Works! (tumhare DNS mein mapping hai)
www.google.com → ❌ Not found! (tumhare DNS mein mapping nahi)

Kyun? Tumhara DNS server sirf tumhari website jaanta hai,
       google.com nahi!

Fix: Testing ke baad DNS settings revert karo!
```

---

### 🔄 DNS Settings Revert karna

#### Mac:
```
System Settings → Network → DNS
→ Woh 10.21.55.1 entry remove karo
→ OK → Normal internet wapas ✅
```

#### resolv.conf revert:
```bash
vi /etc/resolv.conf
# Nameserver ko original value pe wapas karo
# Ya DHCP se automatic ane do
```

---

## 10. DNS Record Types

### 📊 Common DNS Record Types

| Record Type | Full Form | Kya karta hai | Example |
|---|---|---|---|
| **A** | Address Record | Hostname → IPv4 IP | `www → 10.21.55.1` |
| **AAAA** | IPv6 Address | Hostname → IPv6 IP | `www → 2001:db8::1` |
| **NS** | Name Server | Domain → DNS Server | `myweb.com → ns1.myweb.com` |
| **CNAME** | Canonical Name | Hostname → Hostname | `www → mysite.com` |
| **PTR** | Pointer Record | IP → Hostname (reverse) | `10.21.55.1 → www.myweb.com` |
| **MX** | Mail Exchange | Domain → Mail Server | `myweb.com → mail.myweb.com` |
| **SOA** | Start of Authority | Zone metadata | Primary NS, serial, TTL info |
| **TXT** | Text Record | Arbitrary text | Domain verification, SPF |
| **SRV** | Service Record | Service location | VoIP, gaming servers |

---

### 🔄 A Record vs CNAME vs PTR

```
A Record (Forward):
  www.myweb.com → 10.21.55.1
  (Name → IP)

CNAME Record:
  blog.myweb.com → www.myweb.com → 10.21.55.1
  (Alias → Real Name → IP)

PTR Record (Reverse):
  10.21.55.1 → www.myweb.com
  (IP → Name)
  Used for: Email server verification, security
```

---

## 11. Zone File Types

### 📁 2 Types of Zone Files

| Type | Direction | Use |
|---|---|---|
| **Forward Zone** | Domain → IP | Browser se website access karna |
| **Reverse Zone** | IP → Domain | IP se domain pata karna |

---

### 📄 Forward Zone File

```bash
# File naam convention:
/var/named/myweb.com.fzone

# Content:
www    IN    A    10.21.55.1
# Domain → IP
```

---

### 📄 Reverse Zone File (PTR Records ke liye)

**named.conf mein reverse zone block:**

```
zone "55.21.10.in-addr.arpa" IN {
    type master;
    file "myweb.com.rzone";
    allow-query { any; };
};
```

> 💡 `55.21.10.in-addr.arpa` = IP `10.21.55.x` ko reverse mein likha + `.in-addr.arpa`

**Reverse zone file content:**

```
$TTL 2D
@    IN    SOA    ns1.myweb.com.    admin.myweb.com. (
                  800    ; Serial
                  3600   ; Refresh
                  1800   ; Retry
                  604800 ; Expire
                  86400) ; Minimum

@    IN    NS     ns1.myweb.com.
1    IN    PTR    www.myweb.com.
# ↑ Last octet of IP (10.21.55.**1**)
```

---

## 12. Quick Revision

### 🔄 Complete DNS Setup Flow

```
1. Web Server Setup:
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   # /var/www/html/index.html mein content daalo
   firewall-cmd --add-service=http --permanent
   firewall-cmd --reload
   # Test: http://10.21.55.1 ✅

2. DNS Server Setup:
   yum install -y bind bind-utils
   systemctl start named
   systemctl enable named
   firewall-cmd --add-service=dns --permanent
   firewall-cmd --reload

3. /etc/named.conf Edit:
   listen-on → Apni IP add karo
   Zone block add karo (domain + file naam)

4. Zone File banao:
   vi /var/named/myweb.com.fzone
   SOA + NS + A records add karo

5. Verify:
   named-checkconf     (config check)
   named-checkzone myweb.com /var/named/myweb.com.fzone  (zone check)

6. Service Restart:
   systemctl restart named

7. resolv.conf edit:
   nameserver 10.21.55.1

8. Test:
   nslookup www.myweb.com
   → Address: 10.21.55.1 ✅

9. Browser:
   www.myweb.com → Website opens! ✅
```

---

### 📝 Key Terms Glossary

| Term | Matlab |
|---|---|
| **DNS** | Domain Name System — naam se IP dhundta hai |
| **Domain Name** | Human-readable website naam (google.com) |
| **IP Address** | Machine-readable address (142.250.77.46) |
| **DNS Server** | Domain → IP mapping store karta hai |
| **BIND** | Most popular open-source DNS software |
| **named** | BIND DNS service ka process naam |
| **Zone** | Ek domain ke liye DNS data ka collection |
| **Zone File** | Text file jisme DNS records hote hain |
| **Forward Zone** | Domain → IP mapping |
| **Reverse Zone** | IP → Domain mapping |
| **A Record** | Hostname → IPv4 address |
| **NS Record** | Domain ke name servers |
| **SOA Record** | Zone ka master record — authority info |
| **CNAME** | Hostname → Another hostname (alias) |
| **PTR Record** | IP → Hostname (reverse lookup) |
| **MX Record** | Mail exchange server |
| **TTL** | Time To Live — cache validity duration |
| **Serial Number** | Zone file version — change karo har edit pe |
| **resolv.conf** | System ko batata hai kaunsa DNS use karo |
| **nslookup** | DNS query tool — test ke liye |
| **named-checkconf** | named.conf file syntax check tool |
| **named-checkzone** | Zone file syntax check tool |

---

### ⚡ Important Commands Cheat Sheet

```bash
# Web Server
yum install -y httpd               # Install
systemctl start/stop/restart httpd # Control
systemctl status httpd             # Status check
firewall-cmd --add-service=http    # Firewall allow

# DNS Server
yum install -y bind bind-utils     # Install
systemctl start/stop/restart named # Control
systemctl status named             # Status check
firewall-cmd --add-service=dns     # Firewall allow

# Verification
named-checkconf                    # Config file check
named-checkzone domain /path/file  # Zone file check
nslookup domain [dns-server-ip]    # DNS query test

# Firewall
firewall-cmd --add-service=SERVICE --permanent
firewall-cmd --reload
firewall-cmd --list-services       # Active services

# File locations
/etc/named.conf                    # Main DNS config
/var/named/                        # Zone files directory
/etc/resolv.conf                   # System DNS setting
/var/www/html/                     # Web server root
```

---

### ⚠️ Common Mistakes to Avoid

```
❌ Zone file mein domain ke end mein dot (.) nahi lagaya
   Wrong: www    IN    A    10.21.55.1
            ↓ domain naam ke end mein dot missing
   
   Right (full domain names mein):
   ns1.myweb.com.    ← dot at end!

❌ Serial number update nahi kiya zone file edit karne pe
   → DNS cache update nahi hoga

❌ Firewall mein port allow nahi kiya
   → DNS server bahar se accessible nahi hoga

❌ named-checkconf run nahi kiya → Config errors chali gaye
❌ named-checkzone run nahi kiya → Zone file errors
❌ Service restart nahi kiya → Changes apply nahi hue

❌ resolv.conf revert nahi kiya testing ke baad
   → Internet access band ho jaayega!

❌ Config file ka backup nahi liya
   → Galti hui toh wapas original pe nahi ja sakte
```

---

### 🔍 Troubleshooting Guide

```
Problem: Website IP se open ho rahi hai lekin www.myweb.com se nahi

Check 1: resolv.conf mein sahi DNS server hai?
  cat /etc/resolv.conf
  nameserver 10.21.55.1  ← ye hona chahiye

Check 2: Named service running hai?
  systemctl status named

Check 3: Firewall DNS allow hai?
  firewall-cmd --list-services | grep dns

Check 4: Zone file correct hai?
  named-checkzone myweb.com /var/named/myweb.com.fzone

Check 5: nslookup se test karo:
  nslookup www.myweb.com 10.21.55.1

─────────────────────────────────────────
Problem: Internet band ho gayi

Fix: resolv.conf mein apna DNS server remove karo
  vi /etc/resolv.conf → original nameserver pe wapas jaao
```

---

> 🚀 **Next Topics:** SSL/TLS Certificates, HTTPS Setup, Load Balancing, VPC Networking