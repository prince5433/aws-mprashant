# ☁️ AWS Zero to Hero — Hinglish Notes
> **Source:** Chai aur Code — AWS Zero to Hero Course  
> **Format:** Hinglish Markdown Notes  
> **Topics Covered:** Virtualization → Cloud Computing → AWS Overview → IAM → CLI

---

## 📌 Table of Contents

1. [Virtualization kya hota hai?](#1-virtualization-kya-hota-hai)
2. [Cloud Computing kya hai?](#2-cloud-computing-kya-hai)
3. [AWS kya hai?](#3-aws-kya-hai)
4. [AWS Account Setup](#4-aws-account-setup)
5. [IAM — Identity & Access Management](#5-iam--identity--access-management)
6. [AWS CLI Configuration](#6-aws-cli-configuration)

---

## 1. Virtualization kya hota hai?

### 🧠 Concept

Ek normal laptop/desktop mein ye layers hoti hain:

```
Hardware (RAM, CPU, HDD)
    ↓
Operating System (Windows / Mac)
    ↓
Applications
```

**Problem:** Agar tujhe ek hi machine pe multiple OS chahiye (jaise Windows pe Linux bhi chalana ho) — toh kya karein?

**Solution → Virtualization!**

---

### ⚙️ Virtualization kaise kaam karta hai?

```
Physical Machine (Hardware)
        ↓
  [ Hypervisor Layer ]
   /         \
VM-1         VM-2
(Linux)    (Windows)
2GB RAM    2GB RAM
20GB HDD   10GB HDD
```

- **Hypervisor** ek software hai jo virtual machines banata aur manage karta hai
- Ye host machine ke hardware resources ko divide karke **mini computers** banata hai
- In mini computers ko hi **Virtual Machines (VMs)** bolte hain
- Har VM apne **isolated environment** mein run karta hai — ek ka crash doosre ko affect nahi karta

> 💡 **Analogy:** Ghar mein ek bada kamra hai, usme deewarein daalo aur 3 alag rooms bana lo — same space, alag-alag use!

---

### 🔀 Hypervisor ke Types

| Feature | Type 1 (Bare Metal) | Type 2 (Hosted) |
|---|---|---|
| Dusra naam | Bare Metal Hypervisor | Hosted Hypervisor |
| Kahan install hota hai | Directly hardware pe | Host OS ke upar |
| Host OS ki zaroorat | Nahi | Haan |
| Performance | Zyada fast | Thoda slower |
| Examples | VMware vSphere, Citrix XenServer, AWS ka infrastructure | Oracle VirtualBox, VMware Workstation |

> 🏢 **AWS bhi Type 1 Hypervisor use karta hai** — unke paas bade-bade servers hain, jo virtualization se chote-chote VMs mein divide karke different organizations/users ko assign karte hain.

---

### ✅ Companies Virtualization kyun use karti hain?

- **Maintenance headache nahi** — Security updates, OS upgrades sab cloud provider karta hai
- **Cost effective** — Dedicated hardware kharidne ki zaroorat nahi
- **Scalable** — Resources badhana/ghatana aasan hai
- **Isolation** — Ek VM crash ho toh doosri safe rehti hai

---

## 2. Cloud Computing kya hai?

### 📖 Definition

> **"On-demand delivery of IT resources over the Internet with pay-as-you-go pricing"**

Simple bolo toh: **Internet ke zariye IT resources (servers, storage, databases, software) rent pe lena.**

---

### 🏗️ Cloud kaise kaam karta hai?

```
Cloud Provider (AWS / Azure / GCP)
         ↓
   Huge Data Centers
  (Physical Servers + Networking)
         ↓
  Virtualization Technology
         ↓
Multiple Users ko serve karo
```

- Different locations mein inke **bade-bade Data Centers** hain
- Virtualization use karke ek hi infrastructure se **millions of users** ko serve karte hain

---

### 🏪 Cloud ke 3 Service Types (IaaS / PaaS / SaaS)

#### Ice Cream Parlour Analogy 🍦

| Type | Full Form | Analogy | Tu kya manage karta hai |
|---|---|---|---|
| **IaaS** | Infrastructure as a Service | Khali dukaan mili — paint, furniture, machine sab tu laayega | OS, Applications sab tera zimma |
| **PaaS** | Platform as a Service | Dukaan already set hai — furniture, freezer, tray sab ready | Sirf apna product (ice cream) laao |
| **SaaS** | Software as a Service | Vending machine — sirf paisa daalo, ice cream lo | Kuch nahi — bas use karo! |

> 💡 **Real examples:**
> - **IaaS** → AWS EC2 (virtual machine lena)
> - **PaaS** → Heroku, Elastic Beanstalk
> - **SaaS** → Google Docs, Gmail, Notion

---

### 🌍 Cloud Deployment Types

| Type | Meaning | Analogy | Benefit |
|---|---|---|---|
| **Public Cloud** | Shared environment — multiple users ek hi data center use karte hain | Public bus — seat teri, bus sab ki | Sasta, flexible |
| **Private Cloud** | Ek organization ke liye dedicated | Rental car — sirf teri family ke liye | Privacy + Security |
| **Hybrid Cloud** | Mix of public + private | Kuch rasta bus se, kuch taxi se | Best of both worlds |

---

### 🎯 Cloud ke Fayde (Raaju ka use case)

Raaju ne ek e-commerce website banayi — requirements thi:
- 24x7 running
- Secure (credit card data)
- Fast loading
- Backup in case of failure
- Handle sudden traffic spikes

**Problem with buying servers:** Manage karna, update karna, security, hire team — bahut complicated + expensive!

**Cloud Solution:**
- Sab kuch cloud provider handle karta hai
- Pay only for what you use
- Minutes mein scale up/down karo
- Browser dashboard se manage karo

---

## 3. AWS kya hai?

### 📌 Overview

**AWS = Amazon Web Services = Cloud Computing Platform by Amazon**

- **2006** mein launch hua — **duniya ka pehla cloud platform**
- Initially sirf S3 (storage) aur EC2 (compute) the
- Aaj **200+ fully-featured services** provide karta hai
- AI/ML, IoT, Serverless sab kuch cover hai

---

### 📊 Market Position (Q2 2024)

```
AWS        → 32% market share  ← MOST POPULAR
Azure      → ~23%
Google GCP → ~12%
Others     → remaining
```

> AWS = Most widely used cloud platform globally

---

### 🌐 AWS Global Infrastructure

```
World Map pe AWS ka infrastructure:

34 Launched Regions
      ↓
108 Availability Zones
      ↓
Multiple Local Zones (ultra-low latency ke liye)
```

#### Regions vs Availability Zones

| Term | Matlab |
|---|---|
| **Region** | Ek geographic location — jaise Mumbai, Stockholm, US-East |
| **Availability Zone (AZ)** | Region ke andar multiple data centers — single point of failure avoid karne ke liye |

> 💡 **Mumbai region mein 3 AZs hain** — ek building mein power failure aayi toh doosra data center active rahega!

---

### 🛠️ AWS ki Popular Services

| Service | Kaam |
|---|---|
| **EC2** | Virtual machines (Elastic Compute Cloud) |
| **S3** | File/Object Storage |
| **RDS** | Relational Database Service |
| **Lambda** | Serverless computing |
| **CloudFront** | CDN — Content Delivery Network |
| **IAM** | User & Access Management |

---

### 💼 Career Scope after AWS

- Cloud Engineer
- Solutions Architect
- DevOps Engineer
- Cloud Developer
- Data Engineer
- Cloud Security Specialist
- ML Engineer (AWS)
- AWS Support Engineer
- Cloud Consultant

---

### 💰 AWS Free Tier

Pehla account banane pe:
- **12 months free** basic services
- **Always free** kuch services (jaise IAM — bilkul free!)
- **Free trials** — limited time/resources

> Example: EC2 t3.micro — 750 hours/month free for first year!

**Pricing Calculator:** `calculator.aws` pe jao — pehle se estimate karo kitna cost hoga.

---

## 4. AWS Account Setup

### 📋 Kya chahiye pehle?

- ✅ Working email address (verification hogi)
- ✅ Mobile number (OTP verification)
- ✅ Government ID — Aadhar / Voter ID / Driving License
- ✅ Debit/Credit card (verification ke liye ₹2 katenge — baad mein refund)

---

### 🔧 Steps

```
1. aws.amazon.com pe jao
2. "Create a Free Account" click karo
3. Email + Password set karo
4. Credit/Debit card details do
5. Purpose: Personal Use select karo
6. Document verification (Aadhar/DL)
7. Phone number verify karo
8. Basic Support (Free) select karo
9. 🎉 Account ready!
```

---

### 🖥️ AWS Console Overview

Login ke baad dikhega:

- **Recently Visited Services** — jo services use ki hain
- **Region selector** (top-right) — apna region choose karo
  - India mein ho → Mumbai / Hyderabad select karo
- **Cost & Usage** — kitna bill aa raha hai dekhta rehna
- **Services menu** → sab 200+ services yahan milti hain

> 🌑 Pro tip: Dark mode on kar lo — ankhon ko zyada aacha lagega!

---

## 5. IAM — Identity & Access Management

### 📖 Definition

> IAM ek **free, global service** hai jo tumhare AWS account pe **users, groups aur permissions** manage karne deta hai.

---

### 🔑 Root Account vs IAM User

| Feature | Root Account | IAM User |
|---|---|---|
| Powers | Sabse zyada — "mallik" of the account | Sirf assigned permissions |
| Use karo | Sirf initial setup ke liye | Roz ka kaam yahi se karo |
| Risk | Bahut high | Controlled |
| Recommendation | **MFA zaroor enable karo** | Group ke through permissions do |

> ⚠️ **Best Practice:** Root account se roz kaam mat karo. IAM user banao aur use karo.

---

### 👥 Users vs Groups

```
Users:  Alex, Paul, Priya, Rahul (individual identities)

Groups: 
  [Operations Team] → Alex, Paul
  [Dev Team]        → Priya, Rahul

Permissions:
  Operations Group ko → EC2 + S3 access
  Dev Group ko        → Full EC2 access
```

> 💡 **Fayda:** 10-20 users hain toh har user ko alag-alag permission dene ki jagah — **group ko ek baar permission do, sab users ko mil jaayegi!**

---

### 📜 Policies — Permissions ka Format

Policies **JSON format** mein hoti hain:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

- `Effect`: `Allow` ya `Deny`
- `Action`: Kya karne dena hai (`*` = sab kuch)
- `Resource`: Kaunse resources pe (`*` = sab)

**956+ built-in policies** already available hain. Custom bhi bana sakte ho — visual editor ya JSON dono se.

---

### 🔐 MFA — Multi-Factor Authentication

**MFA = Extra security layer**

Normal login: Username + Password (sirf 2 cheez)

MFA ke baad: Username + Password + **OTP from your phone** (3 cheez)

```
Hacker ke paas password aa bhi gaya →
Still can't login without your phone's OTP
→ Account safe! ✅
```

#### MFA Setup kaise karein?

1. IAM Dashboard → Root User → "Add MFA"
2. Device name do (e.g., `my-phone`)
3. **Authenticator App** choose karo
4. **Google Authenticator** ya **Twilio Authy** download karo mobile pe
5. App mein QR code scan karo
6. Do OTP codes enter karo (30 sec gap se)
7. ✅ MFA active!

---

### 🎭 Roles vs Users

| Feature | IAM User | IAM Role |
|---|---|---|
| Kiske liye | Insaan ke liye | AWS services ke liye |
| Permanent | Haan | Nahi — temporary access |
| Example | Alex ko S3 access | EC2 instance ko RDS access dena |

---

### 🛠️ IAM — Practical Steps

#### User banao:
```
IAM → Users → Create User
→ Name: alex
→ Enable console access
→ Custom password set karo
→ Next: Set Permissions
→ Add to Group (ya direct policy attach karo)
→ Create User ✅
```

#### Group banao:
```
IAM → User Groups → Create Group
→ Name: admin
→ Policy attach karo: AdministratorAccess
→ Create Group ✅
→ Phir User ko is group mein add karo
```

#### Access Check karo:
```
1. User ke Console Sign-in Link copy karo
2. Incognito mein open karo
3. Username + Password se login karo
4. Verify karo kya access hai
```

---

### 📊 Credential Report — Audit ke liye

`IAM → Credential Report → Download CSV`

Ek hi file mein milega:
- Har user ka naam
- Password enabled hai ya nahi
- Last kab use hua
- MFA active hai ya nahi
- Access keys ka status

> Regular audit karte raho! Purani/unused permissions clean karo.

---

### ✅ IAM Best Practices — Summary

| Practice | Kyu? |
|---|---|
| Root account avoid karo daily use ke liye | Root compromise = game over |
| Users ko Groups mein add karo | Permission management easy hoti hai |
| MFA enable karo — especially root ke liye | Extra security layer |
| Strong Password Policy set karo | Brute force se bachao |
| Access Keys kabhi share mat karo | Security breach ho sakta hai |
| Regular Credential Report check karo | Unused access clean raho |
| Least Privilege Principle follow karo | Sirf utna access do jitna zaroor ho |

---

## 6. AWS CLI Configuration

### 📖 CLI kya hai aur kyun use karein?

**CLI = Command Line Interface**

| Method | Limitation |
|---|---|
| AWS Console (Dashboard) | Manually present rehna padta hai |
| CloudShell (Browser Terminal) | Browser pe dependent |
| **AWS CLI (Local Terminal)** | ✅ Scripts + Automation possible! |

> 💡 **Best use case:** Raat ko 2 baje automatic backup ya maintenance task run karna ho — CLI se scripts likho aur **automate karo!**

---

### 🔧 CLI Setup — Windows

```
1. aws.amazon.com/cli pe jao
2. Windows 64-bit installer download karo
3. Double click → Next → Next → Install → Finish
4. CMD open karo
5. Type: aws
   → If usage info dikhti hai → ✅ installed!
```

---

### 🔧 CLI Setup — Mac

```bash
# Download karo aur install karo
# Terminal mein check karo:
aws --version
```

---

### 🔑 Access Keys Generate karo

CLI ko AWS account se connect karne ke liye **Access Key + Secret Key** chahiye:

```
IAM → Users → alex → Security Credentials
→ Access Keys → Create Access Key
→ Purpose: CLI
→ Confirm → Create

✅ Milega:
   Access Key ID:     AKIA...
   Secret Access Key: xyz123...

⚠️ SECRET KEY SIRF EK BAAR DIKHTI HAI — CSV download karo!
```

---

### ⚙️ CLI Configure karo

```bash
aws configure
```

```
AWS Access Key ID:     [paste your access key]
AWS Secret Access Key: [paste your secret key]
Default region name:   eu-north-1   # (ya apna region)
Default output format: [Enter — skip]
```

---

### 🧪 Test karo — pehli AWS CLI command

```bash
aws iam list-users
```

Expected output:
```json
{
  "Users": [
    {
      "UserName": "alex",
      "UserId": "AIDA...",
      "CreateDate": "2024-...",
      ...
    }
  ]
}
```

> ✅ Agar yeh output aaya — CLI sahi se configured hai!

---

### 🌐 Ways to Access AWS — Summary Table

| Method | Kaise | Best For |
|---|---|---|
| **Management Console** | Browser Dashboard | Visual exploration, beginners |
| **CloudShell** | Browser Terminal | Quick commands, no setup |
| **AWS CLI** | Local Terminal | Automation, scripting |
| **AWS SDK (CDK)** | Code mein (Python, JS, etc.) | App integration |
| **API** | Direct HTTP calls | Advanced programmatic access |

---

## 🔄 Quick Revision — Flow Diagram

```
Virtualization
    ↓
Hypervisor (Type 1 / Type 2)
    ↓
Virtual Machines (VMs)
    ↓
Cloud Computing (IaaS / PaaS / SaaS)
    ↓
AWS (200+ services, 34 regions, 108 AZs)
    ↓
Account Setup (Email + Card + ID)
    ↓
IAM (Users → Groups → Policies → MFA)
    ↓
CLI (Access Key → aws configure → commands)
```

---

## 📝 Key Terms Glossary

| Term | Matlab |
|---|---|
| **Virtualization** | Software se virtual machines banana |
| **Hypervisor** | VM banane wala software |
| **VM (Virtual Machine)** | Software-based computer |
| **Cloud Computing** | Internet pe IT resources rent pe lena |
| **IaaS** | Blank computer rent pe — sab tera zimma |
| **PaaS** | Platform ready — sirf app laao |
| **SaaS** | Ready software — bas use karo |
| **AWS** | Amazon ka cloud platform |
| **Region** | Geographic location of data centers |
| **Availability Zone** | Region ke andar multiple data centers |
| **IAM** | AWS user & permission management service |
| **Root User** | Account ka mallik — full power |
| **IAM User** | Limited power user |
| **Policy** | JSON mein define ki gayi permissions |
| **MFA** | Multi-Factor Auth — extra security |
| **Access Key** | CLI ke liye programmatic credentials |
| **CLI** | Command Line se AWS access karna |
| **Free Tier** | Pehle 12 months mein free services |

---

> 🚀 **Next Topics (Coming Soon):** EC2 (Virtual Machines on AWS), S3 (Storage), VPC (Networking), Security Groups, Load Balancing