# 🖥️ AWS EC2 — Elastic Cloud Compute | Hinglish Notes
> **Source:** Chai aur Code — AWS Zero to Hero Course  
> **Topic:** EC2 — Virtual Servers in the Cloud  
> **Coverage:** EC2 Basics → Instance Launch → Security Groups → SSH Access → Instance Types → Purchasing Options

---

## 📌 Table of Contents

1. [EC2 kya hai?](#1-ec2-kya-hai)
2. [EC2 Instance Launch karna](#2-ec2-instance-launch-karna)
3. [User Data — Startup Script](#3-user-data--startup-script)
4. [Security Groups — Firewall Rules](#4-security-groups--firewall-rules)
5. [EC2 Instance ko Remote Access karna](#5-ec2-instance-ko-remote-access-karna)
6. [Instance Types](#6-instance-types)
7. [Purchasing Options](#7-purchasing-options)
8. [Billing & Cost Management](#8-billing--cost-management)
9. [Quick Revision](#9-quick-revision)

---

## 1. EC2 kya hai?

### 📖 Definition

> **EC2 = Elastic Cloud Compute**  
> *"A cloud service that provides resizable virtual servers (called Instances)"*

Simple bolo toh: **Cloud pe ek virtual computer rent pe lena** — jisko tum internet ke through access karte ho.

---

### 🧠 Visualize karo

```
Normal scenario:
  Physical Server kharidna → Costly + Maintenance ka jhanjhat

EC2 ke saath:
  Virtual Server → Cloud pe → Pay as you go → No maintenance tension
```

- Server **tumhare saamne nahi hai** — internet pe hai → isliye "Virtual Server"
- Apne personal computer ki tarah **manage + control** kar sakte ho
- Uspe **koi bhi application, website host** kar sakte ho

---

### ⚙️ EC2 mein kya-kya configure kar sakte ho?

| Configuration | Matlab |
|---|---|
| **Operating System** | Linux, Windows, Amazon Linux, RedHat, etc. |
| **Instance Type** | CPU + RAM ki quantity (t3.micro, m5.large, etc.) |
| **AMI** | Amazon Machine Image — OS ka pre-built template |
| **Storage (EBS)** | Hard disk space — kitna chahiye |
| **Key Pair** | SSH se server access karne ke liye |
| **Security Group** | Firewall rules — kaun sa traffic allow/block |
| **Network Settings** | Public IP, VPC, Subnet |
| **IAM Role** | Server ko AWS services access dena |
| **User Data** | Startup pe automatically run hone wali script |
| **Elastic IP** | Static public IP (change nahi hoti) |

---

### 🔑 Important Note — EC2 is Region Specific!

> ⚠️ EC2 instances **region-specific** hote hain!

```
Stockholm region mein banaya instance
    ↓
Mumbai region mein switch kiya
    ↓
Instance DIKHEGA NAHI! 😱
```

**Fix:** Region dropdown (top-right) mein wahi region select karo jahan instance banaya tha.  
**Shortcut:** EC2 → **Global View** use karo — ek hi jagah sab regions ke instances dikhenge!

---

## 2. EC2 Instance Launch karna

### 🚀 Step-by-Step Process

```
AWS Console → Search "EC2" → EC2 Dashboard → Launch Instance
```

---

### Step 1: Name do

```
Name: my-web-server
(koi bhi meaningful naam)
```

---

### Step 2: AMI (Amazon Machine Image) choose karo

AMI = OS ka pre-built template

| AMI | Note |
|---|---|
| **Amazon Linux 2** | ✅ Free tier eligible — recommended for beginners |
| Ubuntu | Popular for web apps |
| Windows Server | Zyada costly |
| RedHat | Enterprise use |

> 💡 **Amazon Linux 2 / Amazon Linux 2023** use karo — free tier + AWS optimized hai

---

### Step 3: Instance Type choose karo

| Type | vCPU | RAM | Free Tier? |
|---|---|---|---|
| **t2.micro** | 1 | 1 GB | ✅ Yes |
| **t3.micro** | 2 | 1 GB | ✅ Yes |
| t3.small | 2 | 2 GB | ❌ No |

> Learning phase mein **t2.micro ya t3.micro** use karo — Free tier mein aata hai!

---

### Step 4: Key Pair create karo

**Key Pair = Server access karne ki "digital chabi"**

```
"Create Key Pair" → Naam do (e.g., my-web-server-key) → RSA → .pem format → Create

✅ Ek .pem file automatically download hogi
   (e.g., my-web-server-key.pem)
```

> ⚠️ **Ye file sirf ek baar milti hai — safely save karo!**  
> Khoi toh dobara generate karni padegi (purani key kaam nahi karegi)

---

### Step 5: Network Settings

```
✅ Auto-assign Public IP: Enable (warna browser se access nahi kar paoge)

Firewall (Security Group):
  ✅ Allow SSH traffic (port 22) — server ko terminal se access karne ke liye
  ✅ Allow HTTP traffic (port 80) — website access ke liye
```

---

### Step 6: Storage

```
Default: 8 GB (General Purpose SSD - gp2/gp3)
Free Tier limit: Up to 30 GB
```

---

### Step 7: Launch!

```
Summary check karo → "Launch Instance" → Done! ✅
```

Instance launch hone ke baad:
- **Instance ID** milegi
- **Public IP** milegi → browser mein paste karo → website open!
- **Private IP** milegi → internal communication ke liye

---

## 3. User Data — Startup Script

### 💡 Concept

> Jab instance **pehli baar start ho**, tab automatically ek script run karwao!

```
Instance create → Chalu hua → User Data script execute hogi → Kaam ho gaya! 🎉
```

### 📝 Example Script — Apache Web Server Install karna

```bash
#!/bin/bash
# System update karo
yum update -y

# Apache web server install karo
yum install -y httpd

# Apache start karo
systemctl start httpd
systemctl enable httpd

# Ek simple HTML page banao
echo "<h1>Welcome to Apache Web Server on Amazon EC2!</h1>" > /var/www/html/index.html
```

### Kahan dalte hain ye script?

```
Launch Instance → Advanced Details → User Data → Script paste karo
```

### Result:
- Instance launch hua
- Automatically Apache install hua
- `http://<Public-IP>` browser mein open karo → **"Welcome to Apache Web Server"** dikhega! 🎉

---

## 4. Security Groups — Firewall Rules

### 📖 Concept

> **Security Group = EC2 ka Firewall**  
> Ye control karta hai ki **kaun sa traffic andar (Inbound) aur bahar (Outbound) jaaye**

```
Internet/Browser
      ↓
 [Security Group]  ← Firewall ka kaam karta hai
 Inbound Rules     ← Bahar se aane wala traffic
 Outbound Rules    ← Server se bahar jaane wala traffic
      ↓
  EC2 Instance
```

---

### 📥 Inbound vs 📤 Outbound

| Type | Matlab | Default |
|---|---|---|
| **Inbound** | Bahar se server pe aane wala traffic | 🔴 Sab blocked (sirf jo allow karo wahi chalega) |
| **Outbound** | Server se bahar jaane wala traffic | 🟢 Sab allowed |

> 💡 **Outbound allowed kyun?**  
> Jab Apache install kiya → server ne internet pe jaake package download kiya → isliye outbound allow hona zaroori hai!

---

### ⚡ Security Group Rules — Kaise kaam karti hain

```
RULE 1: Port 80 (HTTP) Allow → Anyone can access website ✅
RULE 2: Port 22 (SSH) Allow → Terminal se server access ✅

Port 80 DELETE karo → Website 🚫 Access Denied!
Port 80 WAAPAS ADD karo → Website ✅ Back!
```

> **Important:** Security Groups sirf **ALLOW** rules support karti hain — **DENY** nahi kar sakti directly!

---

### 🔢 Common Ports Cheat Sheet

| Port | Protocol | Use |
|---|---|---|
| **22** | SSH | Remote terminal access |
| **80** | HTTP | Normal websites |
| **443** | HTTPS | Secure websites (SSL) |
| **21** | FTP | File transfer |
| **53** | DNS | Domain name resolution |
| **3306** | MySQL | Database |
| **5432** | PostgreSQL | Database |
| **3000** | Custom | Node.js apps (common) |
| **8080** | Custom | Dev servers |

---

### 🛠️ Security Group Rules Edit karna

```
EC2 Dashboard → Instance click karo → "Security" tab
→ Security Group ID pe click karo
→ "Edit Inbound Rules"
→ Add Rule / Delete Rule
→ Save Rules
```

---

### 🧩 Custom Security Group banana

```
EC2 Dashboard → Left sidebar → Network & Security → Security Groups
→ "Create Security Group"
→ Name + Description do
→ Inbound Rules add karo
→ Outbound Rules (default sab allow — change na karo mostly)
→ Create
```

**Fayda:** Ek baar banao → multiple instances mein attach karo!

```
New Instance banate waqt:
  Network Settings → Security Group → "Select existing" → Apna custom group choose karo
```

---

## 5. EC2 Instance ko Remote Access karna

### 🖥️ 3 Tarike

```
Method 1: EC2 Instance Connect (Browser se)
Method 2: PuTTY (Windows)
Method 3: SSH Command (Mac/Linux)
```

---

### Method 1: EC2 Instance Connect (Sabse Aasan)

```
EC2 Dashboard → Instance select karo → "Connect" button
→ "EC2 Instance Connect" tab → Connect

✅ Browser mein hi terminal khul jayega!
```

**Fayda:** Koi extra software nahi chahiye!

---

### Method 2: PuTTY se SSH (Windows)

#### Step 1: PuTTY install karo
```
google.com → "Download PuTTY" → putty.org → 64-bit installer
```

#### Step 2: .pem file ko .ppk mein convert karo
```
PuTTYgen open karo
→ "Load" → All Files → apni .pem file select karo
→ "Save Private Key" → my-web-server-key.ppk save karo
```

#### Step 3: PuTTY mein connect karo
```
PuTTY open karo
→ Host Name: <Public IP of your EC2>
→ Port: 22
→ Connection → SSH → Auth → Credentials
→ "Private key file" → Browse → .ppk file select karo
→ Open

Login as: ec2-user
✅ Connected!
```

#### Shortcut — Username pehle se save karna
```
PuTTY → Session: my-web-server → Load
→ Host Name field mein: ec2-user@<public-ip>
→ Save → Next time directly login!
```

---

### Method 3: SSH Command (Mac/Linux)

```bash
# File permission fix karo (ek baar)
chmod 400 my-web-server-key.pem

# Connect karo
ssh -i "my-web-server-key.pem" ec2-user@<Public-IP>

# Example:
ssh -i "my-web-server-key.pem" ec2-user@54.123.45.67
```

```
Are you sure you want to continue? → yes → ✅ Connected!
```

---

### 🔧 Connected hone ke baad kya kar sakte ho?

```bash
# HTML file edit karo
sudo vi /var/www/html/index.html

# Vi editor mein:
# 'i' press karo → Insert mode
# Edit karo
# 'Esc' → ':wq' → Enter (save and exit)

# Browser mein refresh karo → Changes dikhenge!
```

---

## 6. Instance Types

### 📊 Categories Overview

```
General Purpose     → t3, m5, m6i  (balanced CPU + RAM)
Compute Optimized   → c5, c6i       (zyada CPU power)
Memory Optimized    → r6g, x1e      (zyada RAM)
Accelerated Computing → g5, p3      (GPU — ML/Video)
Storage Optimized   → i3, d2        (fast storage I/O)
```

---

### 🏷️ Naming Convention

```
t  3  .  micro
↑  ↑      ↑
|  |      Size (nano/micro/small/medium/large/xlarge/2xlarge...)
|  Generation (number badha = newer = better)
Family (t=general, m=general, c=compute, r=memory, g=GPU)
```

---

### 📋 Use Case → Instance Type Mapping

| Use Case | Recommended Type | Reason |
|---|---|---|
| **Learning / Testing** | t2.micro / t3.micro | Free tier, low cost |
| **Simple blog / static website** | t3.micro / t3.small | Low traffic, general purpose |
| **E-commerce website** | m5.large / m5.xlarge | Medium load, general purpose |
| **Real-time video streaming** | g5.12xlarge | GPU needed for rendering |
| **Real-time analytics / in-memory DB** | r6g.large | Memory optimized |
| **Machine Learning training** | p3.2xlarge | High-end GPU |

---

### 🔍 t3 Series Comparison (Free Tier range)

| Type | vCPU | RAM | Use |
|---|---|---|---|
| t3.nano | 2 | 0.5 GB | Ultra minimal |
| **t3.micro** | 2 | 1 GB | **Free tier ✅** |
| t3.small | 2 | 2 GB | Slightly more power |
| t3.medium | 2 | 4 GB | Medium workloads |
| t3.large | 2 | 8 GB | Bigger apps |
| t3.xlarge | 4 | 16 GB | Heavy workloads |
| t3.2xlarge | 8 | 32 GB | Very heavy |

> 📍 Official link: `aws.amazon.com/ec2/instance-types/`

---

## 7. Purchasing Options

### 💰 5 Types of Purchasing Options

```
1. On-Demand
2. Reserved Instances
3. Spot Instances
4. Savings Plans
5. Dedicated Host / Dedicated Instances
```

---

### Detailed Comparison Table

| Option | Best For | Cost | Flexibility | Commitment |
|---|---|---|---|---|
| **On-Demand** | Testing, short-term, unpredictable load | Highest | Maximum | None |
| **Reserved** | Steady, predictable long-term workloads | **Up to 75% off** | Low | 1 or 3 years |
| **Spot** | Flexible, fault-tolerant tasks (batch jobs, ML training) | **Up to 90% off** | Medium | None (can be interrupted!) |
| **Savings Plans** | Flexible commitment | Up to 72% off | Medium | 1 or 3 years |
| **Dedicated Host** | Need full hardware control (compliance/licensing) | Highest | Low | 1 or 3 years |

---

### 🧠 Analogies

| Option | Analogy |
|---|---|
| **On-Demand** | OLA/Uber — just book karo, jab chahiye |
| **Reserved** | Gym membership — 1 saal ka liya, zyada discount |
| **Spot** | Last-minute flight ticket — sasta lekin cancel ho sakta hai |
| **Dedicated Host** | Khud ki gaadi kharid lo — full control, full cost |

---

### ⚠️ Spot Instances — Important Warning

```
AWS ke paas extra capacity hai → tum saste mein le sakte ho
AWS ko capacity chahiye → tumhara instance 2 min notice pe band ho sakta hai!
```

> Spot instances **interrupt ho sakte hain** — kabhi bhi!  
> Use karo: Batch processing, ML training, data analysis — jahan restart ho sake.  
> **Avoid karo:** Production websites, databases

---

## 8. Billing & Cost Management

### 💳 Bill Track kaise karo

```
AWS Console → Search "Billing" → Billing and Cost Management
→ Dashboard milega:
   - Month-to-date cost: ₹/$ kitna hua
   - Forecasted cost: Is mahine total kitna ho sakta hai
   - Service-wise breakdown: Kaun si service kitna le rahi hai
```

---

### 🛑 Instance Terminate karo jab kaam na ho!

**Stop vs Terminate:**

| Action | Matlab | Data? |
|---|---|---|
| **Stop** | Shutdown — later restart kar sakte ho | ✅ Data safe |
| **Terminate** | Delete — instance gone forever | ❌ Data delete |
| **Reboot** | Restart | ✅ Data safe |

```
Instance select karo → "Instance State" → Stop / Terminate
```

> 💡 **Best Practice:** Learning ke baad **Terminate** kar do — warna bina use ke bill aata rahega!

---

### 📊 Free Tier Limits (EC2)

| Resource | Free Tier Limit |
|---|---|
| EC2 t2.micro / t3.micro | **750 hours/month** (first 12 months) |
| EBS Storage | **30 GB** (SSD) |
| Data Transfer | 15 GB outbound |
| Elastic IP | 1 free (jab instance running ho) |

> ⚠️ 750 hours = ~31 days → matlab ek instance 24x7 chala sakte ho free mein!  
> Do instances chalaye → 375 hours each → Free tier cross ho sakta hai!

---

## 9. Quick Revision

### 🔄 EC2 Complete Flow

```
1. EC2 Console → "Launch Instance"
2. Name → Amazon Linux AMI → t2.micro (Free Tier)
3. Key Pair create karo → .pem file download
4. Security Group setup karo (SSH + HTTP allow)
5. Storage: 8GB (default, free hai)
6. User Data script add karo (optional — auto-setup)
7. "Launch Instance" → Done!

Verify karo:
  Public IP copy karo → Browser mein paste karo → Website dikhni chahiye!

Remote Access:
  Browser: EC2 Instance Connect
  Windows: PuTTY (.pem → .ppk convert karo)
  Mac/Linux: ssh -i key.pem ec2-user@<IP>

Kaam khatam? → Instance Terminate karo → Bill aana band!
```

---

### 📝 Key Terms Glossary

| Term | Matlab |
|---|---|
| **EC2** | Elastic Compute Cloud — virtual servers |
| **Instance** | Ek virtual server (EC2 ka ek unit) |
| **AMI** | Amazon Machine Image — OS template |
| **Instance Type** | CPU + RAM configuration (t3.micro, etc.) |
| **Key Pair** | SSH authentication ke liye public-private key |
| **Security Group** | EC2 ka firewall — traffic rules |
| **Inbound Rules** | Bahar se aane wala traffic ka control |
| **Outbound Rules** | Server se baane jaane wala traffic ka control |
| **User Data** | Startup pe auto-run hone wali script |
| **Public IP** | Internet se accessible IP |
| **Private IP** | Internal AWS network IP |
| **Elastic IP** | Static public IP — restart pe nahi badlti |
| **EBS** | Elastic Block Store — EC2 ki hard disk |
| **On-Demand** | Pay per hour, no commitment |
| **Reserved** | 1-3 saal ka commitment, up to 75% off |
| **Spot Instance** | Sasta lekin interrupt ho sakta hai |
| **Terminate** | Instance permanently delete karna |
| **Stop** | Instance band karna (restart kar sakte ho) |
| **PuTTY** | Windows pe SSH client |
| **SSH** | Secure Shell — remote terminal access |

---

### ⚡ Common Commands (After SSH)

```bash
# System update
sudo yum update -y

# Apache install
sudo yum install -y httpd

# Apache start
sudo systemctl start httpd
sudo systemctl enable httpd    # boot pe auto-start

# Apache status check
sudo systemctl status httpd

# HTML file edit
sudo vi /var/www/html/index.html

# File dekhna
cat /var/www/html/index.html

# Logs dekhna
sudo tail -f /var/log/httpd/access_log
```

---

> 🚀 **Next Topics:** S3 (Storage), VPC (Networking), Load Balancing, Auto Scaling