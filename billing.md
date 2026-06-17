# 💰 AWS Billing, Organizations & Cost Management | Hinglish Notes
> **Source:** Chai aur Code — AWS Zero to Hero Course  
> **Topic:** AWS Organizations + Billing + Cost Management  
> **Coverage:** AWS Organizations → SCPs → Pricing Calculator → Billing Dashboard → Cost Explorer

---

## 📌 Table of Contents

1. [AWS Organizations kya hai?](#1-aws-organizations-kya-hai)
2. [Organizations Setup karna](#2-organizations-setup-karna)
3. [Service Control Policies (SCPs)](#3-service-control-policies-scps)
4. [AWS Pricing Calculator](#4-aws-pricing-calculator)
5. [Billing & Cost Management Dashboard](#5-billing--cost-management-dashboard)
6. [Cost Explorer](#6-cost-explorer)
7. [Cost Control Best Practices](#7-cost-control-best-practices)
8. [Quick Revision](#8-quick-revision)

---

## 1. AWS Organizations kya hai?

### 📖 Problem Statement

```
Scenario:
  Team mein 5-6 members hain
  Har member ka alag AWS account hai
  Sab ek hi company/team ke liye kaam karte hain

Problems without Organizations:
  ❌ Har account ka bill alag-alag track karna
  ❌ Har account pe policies alag-alag apply karni padti hain
  ❌ Kisi ne galat service start ki — pata hi nahi chala
  ❌ Admin ko har account mein jaake manage karna padta hai
```

**Solution → AWS Organizations!**

---

### 📖 Definition

> **AWS Organizations = Multiple AWS accounts ko ek jagah se centrally manage karne ki service**

```
AWS Organizations
      ↓
  Root Account (Admin/Management Account)
  ├── Account 1 (Team Member A)
  ├── Account 2 (Team Member B)
  ├── OU: Dev Team
  │   ├── Account 3 (Developer 1)
  │   └── Account 4 (Developer 2)
  └── OU: Finance Team
      └── Account 5 (Finance person)
```

---

### ✅ Key Features

| Feature | Matlab |
|---|---|
| **Central Management** | Ek admin — saare accounts manage karo |
| **Consolidated Billing** | Saare accounts ka bill ek jagah |
| **Service Control Policies (SCP)** | Accounts pe restrict karo kaun si services use ho sakti hain |
| **Organization Units (OU)** | Accounts ko groups mein organize karo |
| **Cost Free** | AWS Organizations use karna **bilkul FREE** hai! |
| **Global Service** | Region-specific nahi — globally available |

> 💡 **Best Part:** AWS Organizations **free hai** — sirf underlying AWS services ka bill aata hai!

---

## 2. Organizations Setup karna

### 🏗️ Structure Samjho

```
Management Account (Root)
  ↓
Organization Units (OUs) — optional groupings
  ↓
Member Accounts — individual AWS accounts

Example:
  Root (Admin Account)
  ├── OU: Production
  │   ├── Account: prod-app
  │   └── Account: prod-db
  ├── OU: Development
  │   ├── Account: dev-frontend
  │   └── Account: dev-backend
  └── Account: billing-account
```

---

### 🛠️ Organization Create karna

```
AWS Console → Search "AWS Organizations" → Click karo

→ "Create Organization" → Enable karo

✅ Tumhara current account automatically Management Account ban jaata hai!
```

---

### ➕ New Account Add karne ke 2 Tarike

#### Tarika 1: Naya Account Create karo

```
AWS Organizations → AWS Accounts → "Add an AWS Account"
→ "Create an AWS Account"
→ Account name do
→ Email address do (unique hona chahiye)
→ IAM Role name (optional)
→ Create ✅

New account automatically organization mein add ho jaata hai!
```

#### Tarika 2: Existing Account ko Invite karo

```
AWS Organizations → AWS Accounts → "Add an AWS Account"
→ "Invite an existing AWS Account"
→ Email address ya Account ID enter karo
→ Send Invitation

Team member ke account mein:
  AWS Organizations → "Invitations"
  → Accept karo → Organization mein join! ✅
```

---

### 🏷️ Organization Units (OUs) banana

```
AWS Organizations → AWS Accounts → Root select karo
→ "Actions" → "Create new Organizational Unit"
→ Name do (e.g., "DevTeam", "Production")
→ Create OU

Accounts move karo:
  Account select karo → Actions → Move → OU select karo
```

---

### 👁️ Organizations Dashboard

```
Management Account → AWS Organizations → AWS Accounts

Dikhega:
  Root
  └── Management Account: [tumhara naam] ← Admin
      └── Member Accounts: (jo join kiye hain)
```

---

## 3. Service Control Policies (SCPs)

### 📖 SCPs kya hain?

> **SCPs = JSON policies jo decide karti hain ki member accounts kaun si AWS services use kar sakte hain (ya nahi kar sakte)**

```
Without SCP: Member account koi bhi service use kar sakta hai
With SCP:    Admin restrict kar sakta hai — "Ye account sirf EC2 use kar sakta hai, S3 nahi!"
```

---

### ⚡ SCPs Enable karna

```
AWS Organizations → Policies → Service Control Policies
→ "Enable Service Control Policies" → Enable karo

Default policy milegi: FullAWSAccess
(Matlab: Admin ke paas sab kuch access hai)
```

---

### 🛠️ Custom SCP banana — Example: S3 Block karna

```
AWS Organizations → Policies → Service Control Policies
→ "Create Policy"

Policy Name: DenyS3Access
Description: S3 access block karo member accounts ke liye
```

**JSON Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

Ya **Visual Editor** use karo:

```
Create Policy → Visual mode
→ Service: S3
→ Actions: All S3 actions
→ Effect: Deny
→ Resource: All
→ Create Policy ✅
```

---

### 🔗 SCP Attach karna

```
SCP create hone ke baad:
  Policy select karo → "Attach" → OU ya Account select karo → Attach

Result:
  Us account ke users S3 access karne ki koshish karein
  → Access Denied! ❌
```

---

### 📊 SCP Rules — Important Points

| Rule | Matlab |
|---|---|
| **SCP sirf member accounts pe apply hoti hai** | Management account pe SCP apply nahi hoti |
| **SCP + IAM dono zaroori hain** | SCP allow kare + IAM allow kare — tabhi access milega |
| **Deny always wins** | Agar SCP deny kare toh IAM allow bhi kaam nahi karega |
| **Hierarchy mein flow hoti hai** | OU pe SCP lagayi → sab child accounts affected |

---

### 🔄 SCP vs IAM — Difference

```
IAM:
  - Individual user/role ke liye permissions
  - Ek account ke andar
  - "Is user ko kya karne dena hai"

SCP:
  - Entire AWS Account ke liye guardrails
  - Organization level se
  - "Is account mein kaun si services allowed hain"

Combined Effect:
  User kuch kar sakta hai ONLY IF:
    ✅ IAM allows it  AND
    ✅ SCP allows it
```

---

## 4. AWS Pricing Calculator

### 📖 Kya hai?

> **Pricing Calculator = AWS resources use karne se PEHLE cost estimate karne ka tool**

```
URL: calculator.aws
```

> 💡 **Use kab karo:** Project planning ke time — boss ya client ko batana ho ki itna budget chahiye!

---

### 🛠️ Estimate banana — Step by Step

```
calculator.aws → "Create Estimate"

Step 1: Region choose karo
  (Jahan deploy karna chahte ho — us region ki pricing different hogi)

Step 2: Service add karo
  "Find Service" → EC2 type karo → Select

Step 3: EC2 Configure karo:
  - Operating System: Linux
  - Instance Type: t3.micro
  - Usage: 730 hours/month (24x7)
  - Pricing: On-Demand

Step 4: More services add karo (optional)
  - RDS, S3, CloudFront, etc.

Step 5: "View Summary"
  → Monthly Cost: $X.XX
  → Total 12-month cost: $X.XX
```

---

### 📊 Estimate Example

```
Simple Web App Setup:
  EC2 t3.micro (1 instance, 730 hrs/month): ~$8.50/month
  RDS db.t3.micro (MySQL):                  ~$25.00/month
  S3 (50 GB storage):                       ~$1.15/month
  Route 53 (1 hosted zone):                 ~$0.50/month
  Data Transfer (10 GB/month):              ~$0.90/month
  ─────────────────────────────────────────────────────
  Total Estimate:                           ~$36.05/month
```

---

### 💡 Calculator ke Faayde

```
✅ Project planning mein budget decide karo
✅ Client ko accurate estimate do
✅ On-Demand vs Reserved compare karo
✅ Different regions ki pricing compare karo
✅ Estimate save karke share kar sakte ho
```

---

## 5. Billing & Cost Management Dashboard

### 📍 Kahan milega?

```
AWS Console → Search "Billing" → "Billing and Cost Management"
Ya: Top-right menu → Account name → Billing Dashboard
```

---

### 🖥️ Dashboard Overview

```
Billing Dashboard mein milega:

┌─────────────────────────────────────────────┐
│  Month-to-date Cost:        $2.47           │
│  Last Month Total:          $1.83           │
│  Forecasted Monthly Bill:   $3.10           │
├─────────────────────────────────────────────┤
│  Top Services by Cost:                      │
│    EC2:     $1.20  ████████                 │
│    Route53: $0.80  █████                    │
│    S3:      $0.30  ██                       │
│    Others:  $0.17  █                        │
└─────────────────────────────────────────────┘
```

---

### 📋 Dashboard ke Sections

| Section | Kya milega |
|---|---|
| **Overview** | Current month ka total cost + forecast |
| **Bills** | Detailed itemized bills — service-wise |
| **Payments** | Pending payments, payment history |
| **Credits** | AWS credits (agar koi hain) |
| **Cost & Usage Reports** | Detailed CSV reports download karo |
| **Cost Explorer** | Interactive graphs + analysis |
| **Budgets** | Budget alerts set karo |
| **Free Tier** | Free tier usage track karo |

---

### 💳 Bills Section

```
Billing → Bills → Month select karo

Dikhega service-wise breakdown:
  Amazon EC2
    ├── Running Hours: 720 hours @ $0.0116/hr = $8.35
    ├── EBS Storage: 8 GB @ $0.10/GB = $0.80
    └── Data Transfer: 1 GB @ $0.09/GB = $0.09
  
  Amazon S3
    └── Storage: 50 GB @ $0.023/GB = $1.15
  
  Amazon Route 53
    └── Hosted Zone: 1 @ $0.50 = $0.50
```

---

### 🔔 Budget Alerts Set karna (Very Important!)

```
Billing → "Budgets" → "Create Budget"

Budget Type: Cost Budget

Settings:
  Budget Name: Monthly AWS Budget
  Period: Monthly
  Budget Amount: $10 (ya jo bhi tumhara limit ho)
  
Alert 1: When actual cost > 80% of budget → Email bhejo
Alert 2: When forecasted cost > 100% of budget → Email bhejo

Email: tumhara email address

→ Create Budget ✅
```

> 🚨 **MUST DO for beginners!** Budget alerts se surprise bills se bacho!

---

### 📊 Free Tier Usage Monitor karna

```
Billing → "Free Tier"

Dikhega:
  Service          | Free Tier Limit    | Your Usage    | Status
  ─────────────────────────────────────────────────────────────
  EC2 t2.micro     | 750 hours/month    | 250 hours     | ✅ Safe
  S3 Storage       | 5 GB               | 2.3 GB        | ✅ Safe
  Lambda           | 1M requests/month  | 10K requests  | ✅ Safe
  RDS db.t2.micro  | 750 hours/month    | 0 hours       | ✅ Safe
```

---

## 6. Cost Explorer

### 📖 Kya hai?

> **Cost Explorer = AWS costs ko visually analyze karne ka tool — graphs, filters, reports sab milta hai**

```
Billing → "Cost Explorer" → "Launch Cost Explorer"
```

---

### 🖥️ Cost Explorer Features

```
1. Time-based graphs:
   Daily, monthly, yearly cost trends

2. Service-wise breakdown:
   Kaun si service kitna le rahi hai

3. Region-wise breakdown:
   Kaun se region mein zyada spend ho raha hai

4. Filtering:
   Service, Region, Account, Tag se filter karo

5. Reports:
   Custom reports save aur share karo

6. Forecast:
   Agle mahine ka estimated bill
```

---

### 📊 Cost Explorer mein kya dekhein?

```
Cost Explorer → Service tab:

  ┌────────────────────────────────────────┐
  │ Total: 19 services used this month    │
  │                                        │
  │ EC2:        $8.35  ████████████       │
  │ RDS:        $4.20  ████████           │
  │ S3:         $1.15  ██                 │
  │ Route 53:   $0.50  █                  │
  │ Others:     $0.80  █                  │
  └────────────────────────────────────────┘

Click on any service → Detailed breakdown!
```

---

### 🔍 Unexpected Bill aaya? Investigation karo!

```
Step 1: Billing Dashboard → Cost breakdown → Konsi service?
Step 2: Cost Explorer → Us service ka daily breakdown dekho
Step 3: Specific din pe spike tha? Kya run tha us din?
Step 4: Service ke dashboard pe jaao → Running resources check karo
Step 5: Unused resources terminate/delete karo
Step 6: Budget alert set karo agar nahi set kiya
```

---

## 7. Cost Control Best Practices

### 💡 Har Beginner ko Follow karna Chahiye

#### 1. Budget Alerts Set Karo (Day 1!)

```
Billing → Budgets → Create Budget
→ $5-10 monthly limit set karo
→ Email alerts enable karo at 80% and 100%
```

#### 2. Free Tier Monitor Karo

```
Billing → Free Tier → Regular check karo
Agar koi service 80%+ use ho gayi → Use karna kam karo ya delete karo
```

#### 3. Resources Cleanup Karo

```
After every practice session:
  ✅ EC2 instances → Terminate (not just Stop!)
  ✅ RDS instances → Delete
  ✅ Elastic IPs → Release (stopped instance pe bhi bill aata hai!)
  ✅ NAT Gateways → Delete (bahut expensive!)
  ✅ Load Balancers → Delete
  ✅ S3 → Empty + Delete (agar test bucket tha)
```

#### 4. EC2 Global View se Check Karo

```
EC2 → Global View → All regions mein instances dekho
Koi chhutti hui instance toh immediate terminate karo!
```

#### 5. Billing Alerts Enable Karo

```
Billing → Billing Preferences → 
  ✅ Receive Free Tier Usage Alerts
  ✅ Receive Billing Alerts (CloudWatch)
  → Save
```

#### 6. IAM User Use Karo (Root nahi)

```
Root account se roz kaam mat karo
IAM user banao → Use karo
Root sirf account setup aur emergency ke liye
```

---

### 🚨 Common Billing Surprises (Aur Bachne ke Tips)

| Surprise | Reason | Bachne ka tarika |
|---|---|---|
| **EC2 bill aata hai** | Instance stop kiya, terminate nahi | Always terminate karo |
| **Elastic IP bill** | Instance stop hai par IP release nahi ki | Release Elastic IP |
| **NAT Gateway** | Delete karna bhool gaye | Practice ke baad delete karo |
| **RDS bill** | DB instance running hai | Delete karo (snapshot optional) |
| **Data Transfer** | Bahut zyada data transfer | CloudFront use karo |
| **Unused S3** | Old test buckets | Empty + delete karo |
| **Route 53** | Hosted zone active hai | Delete records + zone |

---

### 📱 AWS Mobile App se Monitor Karo

```
"AWS Console" app download karo (iOS/Android)

Features:
  - Real-time cost monitoring
  - Push notifications for billing alerts
  - Service health check
  - EC2 instance management
```

---

## 8. Quick Revision

### 🔄 AWS Organizations Flow

```
1. Management Account mein Organizations enable karo
2. Member accounts invite karo ya create karo
3. OUs banao (Dev, Prod, Finance, etc.)
4. Accounts ko OUs mein organize karo
5. SCPs banao aur OUs/accounts pe attach karo
6. Centrally sab manage karo!

Cost: FREE ✅
```

---

### 🔄 Billing Management Flow

```
New AWS Account → Immediately karo:
  1. Budget Alert set karo ($5-10 limit)
  2. Free Tier alerts enable karo
  3. Billing alerts enable karo

Regular Practice ke baad:
  4. Resources cleanup karo
  5. Billing Dashboard check karo
  6. Free Tier usage monitor karo

Unexpected bill aaya:
  7. Cost Explorer → Service-wise breakdown
  8. Offending service identify karo
  9. Unused resources terminate karo
  10. Future ke liye budget alert tighten karo
```

---

### 📝 Key Terms Glossary

| Term | Matlab |
|---|---|
| **AWS Organizations** | Multiple AWS accounts centrally manage karna |
| **Management Account** | Organizations ka admin/root account |
| **Member Account** | Organization mein joined account |
| **OU (Organization Unit)** | Accounts ka logical grouping |
| **SCP (Service Control Policy)** | Account-level permissions restrict karne ki policy |
| **Consolidated Billing** | Saare accounts ka bill ek jagah |
| **Pricing Calculator** | Pehle se cost estimate karne ka tool |
| **Billing Dashboard** | Current + forecasted cost dekhne ka dashboard |
| **Cost Explorer** | Visual cost analysis tool |
| **Free Tier** | Pehle 12 months mein free services |
| **Budget Alert** | Defined limit exceed hone pe email notification |
| **Forecasted Cost** | Is mahine total kitna hoga — prediction |
| **Cost Breakdown** | Service-wise bill detail |
| **Elastic IP** | Static public IP — stopped instance pe bhi bill aata hai! |
| **NAT Gateway** | Private subnet internet access — expensive! |

---

### ⚠️ Golden Rules for AWS Billing

```
Rule 1: Jo banaya, kaam ke baad DELETE karo
Rule 2: Budget alert = AWS account banane ka PEHLA kaam
Rule 3: EC2 "Stop" ≠ "Terminate" — Stop se bhi EBS ka bill aata hai
Rule 4: Elastic IP = Stopped instance pe bhi bill!
Rule 5: NAT Gateway = Bahut expensive — practice ke baad delete karo
Rule 6: Global View use karo — koi instance miss na ho
Rule 7: Free Tier ke liye bhi budget alert set karo
Rule 8: Organizations se team ka cost ek jagah track karo
```

---

### 💡 Useful Links

```
Pricing Calculator:  calculator.aws
Billing Dashboard:   AWS Console → Billing
Cost Explorer:       Billing → Cost Explorer
Free Tier details:   aws.amazon.com/free
Organizations:       AWS Console → Organizations
```

---

> 🚀 **Next Topics:** AWS Amplify, VPC (Virtual Private Cloud), Load Balancer, Auto Scaling