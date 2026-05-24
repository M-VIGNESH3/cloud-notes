# Cloud Computing Basics & Core Platform Concepts

## Complete Implementation Guide (Manual Process)

---

## Table of Contents

1. [Cloud Computing Fundamentals](#1-cloud-computing-fundamentals)
2. [Service Models Deep Dive](#2-service-models-deep-dive)
3. [Deployment Models](#3-deployment-models)
4. [Core Cloud Benefits](#4-core-cloud-benefits)
5. [AWS Core Concepts & Manual Setup](#5-aws-core-concepts--manual-setup)
6. [Azure Core Concepts & Manual Setup](#6-azure-core-concepts--manual-setup)
7. [AWS vs Azure Architecture Comparison](#7-aws-vs-azure-architecture-comparison)
8. [Networking Fundamentals](#8-networking-fundamentals)
9. [Storage Concepts](#9-storage-concepts)
10. [Compute Resources](#10-compute-resources)
11. [Security & Identity Management](#11-security--identity-management)
12. [Monitoring & Management](#12-monitoring--management)
13. [Real-World Architecture Patterns](#13-real-world-architecture-patterns)

---

## 1. Cloud Computing Fundamentals

### What is Cloud Computing?

```
Traditional IT Model:
┌─────────────────────────────────────────────┐
│              Your Data Center               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Physical │  │ Physical │  │ Physical │  │
│  │ Server 1 │  │ Server 2 │  │ Server 3 │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│  ┌──────────────────────────────────────┐   │
│  │         Physical Network             │   │
│  └──────────────────────────────────────┘   │
│  YOU manage everything: hardware, cooling,  │
│  power, security, maintenance               │
└─────────────────────────────────────────────┘

Cloud Model:
┌─────────────────────────────────────────────┐
│           Cloud Provider Data Center        │
│  ┌──────────────────────────────────────┐   │
│  │  Thousands of Physical Servers       │   │
│  │  ┌────┐┌────┐┌────┐┌────┐┌────┐    │   │
│  │  │    ││    ││    ││    ││    │    │   │
│  │  └────┘└────┘└────┘└────┘└────┘    │   │
│  └──────────────────────────────────────┘   │
│  Provider manages: hardware, cooling,       │
│  power, physical security                   │
└──────────────────┬──────────────────────────┘
                   │ Internet
                   ▼
┌─────────────────────────────────────────────┐
│                 YOU Get:                    │
│  Virtual Machines, Storage, Databases,      │
│  Networking - ON DEMAND - PAY AS YOU GO     │
└─────────────────────────────────────────────┘
```

### Five Essential Characteristics (NIST Definition)

```
┌─────────────────────────────────────────────────────────────────┐
│                  5 Essential Cloud Characteristics              │
├─────────────────┬───────────────────────────────────────────────┤
│ Characteristic  │ What It Means                                 │
├─────────────────┼───────────────────────────────────────────────┤
│ On-Demand       │ Resources available instantly without human   │
│ Self-Service    │ interaction from provider                     │
├─────────────────┼───────────────────────────────────────────────┤
│ Broad Network   │ Access from anywhere via internet using       │
│ Access          │ standard protocols (HTTP, HTTPS)              │
├─────────────────┼───────────────────────────────────────────────┤
│ Resource        │ Provider pools computing resources serving    │
│ Pooling         │ multiple customers (multi-tenancy)            │
├─────────────────┼───────────────────────────────────────────────┤
│ Rapid           │ Scale up or down quickly, sometimes           │
│ Elasticity      │ automatically based on demand                 │
├─────────────────┼───────────────────────────────────────────────┤
│ Measured        │ Pay only for what you use, like a utility     │
│ Service         │ bill (electricity, water)                     │
└─────────────────┴───────────────────────────────────────────────┘
```

---

## 2. Service Models Deep Dive

### The Three Cloud Service Models

```
┌──────────────────────────────────────────────────────────────────┐
│                    Cloud Service Model Stack                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    SaaS                                    │  │
│  │           Software as a Service                            │  │
│  │    Gmail, Office 365, Salesforce, Zoom                     │  │
│  │    YOU manage: Nothing (just use the app)                  │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    PaaS                                    │  │
│  │           Platform as a Service                            │  │
│  │    AWS Elastic Beanstalk, Azure App Service                │  │
│  │    YOU manage: Your application code & data                │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    IaaS                                    │  │
│  │         Infrastructure as a Service                        │  │
│  │    AWS EC2, Azure Virtual Machines                         │  │
│  │    YOU manage: OS, middleware, runtime, apps, data         │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ON-PREMISES: YOU manage everything                              │
└──────────────────────────────────────────────────────────────────┘
```

### Responsibility Matrix

```
┌───────────────────┬─────────────┬──────────┬──────────┬──────────┐
│   Component       │ On-Premises │   IaaS   │   PaaS   │   SaaS   │
├───────────────────┼─────────────┼──────────┼──────────┼──────────┤
│ Applications      │     YOU     │   YOU    │   YOU    │ Provider │
│ Data              │     YOU     │   YOU    │   YOU    │ Provider │
│ Runtime           │     YOU     │   YOU    │ Provider │ Provider │
│ Middleware        │     YOU     │   YOU    │ Provider │ Provider │
│ OS                │     YOU     │   YOU    │ Provider │ Provider │
│ Virtualization    │     YOU     │ Provider │ Provider │ Provider │
│ Servers           │     YOU     │ Provider │ Provider │ Provider │
│ Storage           │     YOU     │ Provider │ Provider │ Provider │
│ Networking        │     YOU     │ Provider │ Provider │ Provider │
│ Physical Security │     YOU     │ Provider │ Provider │ Provider │
│ Facilities        │     YOU     │ Provider │ Provider │ Provider │
└───────────────────┴─────────────┴──────────┴──────────┴──────────┘
```

---

## 3. Deployment Models

### Four Cloud Deployment Types

```
┌──────────────────────────────────────────────────────────────────┐
│                      Public Cloud                                │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  AWS / Azure / Google Cloud                              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │    │
│  │  │Company A │  │Company B │  │Company C │              │    │
│  │  │Resources │  │Resources │  │Resources │              │    │
│  │  └──────────┘  └──────────┘  └──────────┘              │    │
│  │  Shared infrastructure, logically separated              │    │
│  └──────────────────────────────────────────────────────────┘    │
│  ✅ Cost-effective  ✅ No maintenance  ✅ Global scale             │
│  ❌ Less control   ❌ Shared environment                          │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      Private Cloud                               │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  Your Organization Only                                  │    │
│  │  ┌──────────────────────────────────────────────────┐    │    │
│  │  │           YOUR dedicated hardware                │    │    │
│  │  │     (on-premises OR hosted by provider)          │    │    │
│  │  └──────────────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────────────┘    │
│  ✅ Full control  ✅ Better security  ✅ Compliance               │
│  ❌ Expensive    ❌ You manage hardware                           │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      Hybrid Cloud                                │
│  ┌────────────────┐          ┌───────────────────────┐           │
│  │ Private Cloud  │◄────────►│    Public Cloud       │           │
│  │ (On-Premises)  │  Secure  │   (AWS / Azure)       │           │
│  │                │  Link    │                       │           │
│  │ Sensitive data │          │ Burst workloads       │           │
│  │ Legacy systems │          │ Dev/Test environments │           │
│  └────────────────┘          └───────────────────────┘           │
│  ✅ Flexibility  ✅ Keep sensitive data local                     │
│  ❌ Complex      ❌ Requires secure connectivity                  │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      Multi-Cloud                                 │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐         │
│  │     AWS      │   │    Azure     │   │ Google Cloud │         │
│  │              │   │              │   │              │         │
│  │ Best-of-breed│   │ Best-of-breed│   │ Best-of-breed│         │
│  │ services     │   │ services     │   │ services     │         │
│  └──────────────┘   └──────────────┘   └──────────────┘         │
│  ✅ No vendor lock-in  ✅ Best services from each                │
│  ❌ Very complex      ❌ Multiple skills needed                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Core Cloud Benefits

### Business Value Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cloud Computing Benefits                     │
│                                                                 │
│  💰 FINANCIAL BENEFITS                                          │
│  ├── Trade CapEx for OpEx                                       │
│  │   Traditional: Buy servers upfront ($50,000)                │
│  │   Cloud: Pay monthly based on usage ($500/month)            │
│  │                                                             │
│  ├── Economies of Scale                                         │
│  │   AWS/Azure buy millions of servers = cheaper per unit      │
│  │   Savings passed to customers                               │
│  │                                                             │
│  └── Stop Guessing Capacity                                     │
│      Traditional: Buy for peak (waste $ at normal times)       │
│      Cloud: Scale to exactly what you need                      │
│                                                                 │
│  ⚡ SPEED & AGILITY BENEFITS                                    │
│  ├── Deploy in minutes instead of weeks                        │
│  ├── Experiment without risk                                    │
│  └── Go global in minutes                                       │
│                                                                 │
│  🔧 OPERATIONAL BENEFITS                                        │
│  ├── Stop spending money on data center operations              │
│  ├── Focus on YOUR applications, not infrastructure             │
│  └── Provider maintains hardware, security patches             │
│                                                                 │
│  🌍 RELIABILITY BENEFITS                                        │
│  ├── Multiple Availability Zones                                │
│  ├── Built-in redundancy                                        │
│  └── Global content delivery                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Cost Model Comparison

```
Traditional IT Cost Over Time:
Cost│                                          
    │    ████                                  
    │    ████  ████                            
    │    ████  ████  ████                      
    │    ████  ████  ████  ████                
    └────Year1─Year2─Year3─Year4─────────────▶
         Big upfront hardware purchases        
         (servers depreciate, still need new ones)

Cloud Cost Over Time:
Cost│                                          
    │ ▓▓ ▓▓ ▓▓▓ ▓▓ ▓▓▓▓ ▓▓▓ ▓▓▓▓▓           
    │ ▓▓ ▓▓ ▓▓▓ ▓▓ ▓▓▓▓ ▓▓▓ ▓▓▓▓▓           
    └────────────────────────────────────────▶
         Pay for what you use each month      
         Scale up/down based on actual need   
```

---

## 5. AWS Core Concepts & Manual Setup

### AWS Global Infrastructure

```
AWS Global Infrastructure:
┌─────────────────────────────────────────────────────────────────┐
│                      AWS Global Map                             │
│                                                                 │
│  Regions: 33+ geographic locations worldwide                    │
│  Availability Zones: 105+ data centers                         │
│  Edge Locations: 400+ for CloudFront CDN                        │
│                                                                 │
│  Region Example: US East (N. Virginia) = us-east-1             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Region (us-east-1)                   │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │    │
│  │  │     AZ      │  │     AZ      │  │     AZ      │    │    │
│  │  │ us-east-1a  │  │ us-east-1b  │  │ us-east-1c  │    │    │
│  │  │             │  │             │  │             │    │    │
│  │  │ Data Center │  │ Data Center │  │ Data Center │    │    │
│  │  │ (separate   │  │ (separate   │  │ (separate   │    │    │
│  │  │  building,  │  │  building,  │  │  building,  │    │    │
│  │  │  power,     │  │  power,     │  │  power,     │    │    │
│  │  │  cooling)   │  │  cooling)   │  │  cooling)   │    │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘    │    │
│  │  Connected by high-speed private fiber                  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create Your First AWS Account

```
STEP 1: Navigate to AWS Account Creation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Open browser → go to: https://aws.amazon.com
2. Click "Create an AWS Account" (top right corner)

STEP 2: Provide Root User Email
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Enter your email address (this becomes ROOT USER)
2. Choose an AWS account name
   ⚠️  IMPORTANT: Root user has FULL access to everything
   ⚠️  After setup, create IAM users for daily use

STEP 3: Email Verification
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Check email for verification code
2. Enter 6-digit code in AWS verification box
3. Click "Verify"

STEP 4: Root User Password
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Create strong password (min 8 chars)
   ✅ Must include: uppercase, lowercase, number, symbol
2. Confirm password
3. Click "Continue"

STEP 5: Contact Information
━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Select account type:
   □ Business  □ Personal
2. Fill in:
   - Full name
   - Phone number
   - Country/Region
   - Address, City, State, Postal code
3. Check "AWS Customer Agreement" box
4. Click "Continue"

STEP 6: Payment Information
━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Enter credit/debit card details
   ℹ️  Required but won't be charged for free tier usage
   ℹ️  AWS free tier gives 12 months free on many services
2. Billing address: same as above or new address
3. Click "Verify and Continue"

STEP 7: Identity Verification
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Choose verification method:
   □ Text message (SMS)
   □ Voice call
2. Enter phone number
3. Enter CAPTCHA characters shown
4. Click "Send SMS" or "Call Me"
5. Enter received verification code
6. Click "Continue"

STEP 8: Select Support Plan
━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Choose plan:
   ┌─────────────────────────────────────────────┐
   │ ● Basic Support (FREE) ← Select this        │
   │   - Account/billing questions               │
   │   - Service limit increases                 │
   │   - AWS documentation access               │
   ├─────────────────────────────────────────────┤
   │ ○ Developer ($29/month)                     │
   │   + Business hours email support            │
   ├─────────────────────────────────────────────┤
   │ ○ Business ($100/month minimum)             │
   │   + 24/7 phone, email, chat support         │
   └─────────────────────────────────────────────┘
2. Click "Complete Sign Up"

STEP 9: Account Activation
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Wait 24-48 hours for activation email
2. Usually activates within minutes to a few hours
3. You'll receive email: "Your AWS Account is Ready"

STEP 10: First Login & Console Navigation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Go to: https://console.aws.amazon.com
2. Sign in as Root user
3. Enter root email → password
4. You're now in AWS Management Console
```

### Understanding AWS Console Layout

```
AWS Management Console Layout:
┌─────────────────────────────────────────────────────────────────┐
│ [≡] aws  [Search services...]          Region▼  Account▼  Bell  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Recently Visited Services:                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │   EC2    │ │   S3     │ │   RDS    │ │  Lambda  │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│                                                                 │
│  All Services (grouped by category):                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Compute:    EC2, Lambda, ECS, EKS, Elastic Beanstalk    │   │
│  │ Storage:    S3, EBS, EFS, Glacier                       │   │
│  │ Database:   RDS, DynamoDB, ElastiCache, Redshift        │   │
│  │ Networking: VPC, Route 53, CloudFront, API Gateway      │   │
│  │ Security:   IAM, KMS, GuardDuty, Shield                 │   │
│  │ Management: CloudWatch, CloudTrail, Config              │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: AWS IAM Setup (Security Best Practice)

```
WHY IAM USERS? Never use root account for daily work!
Root account = master key to your entire AWS account
IAM users = limited keys for specific tasks

STEP 1: Navigate to IAM
━━━━━━━━━━━━━━━━━━━━━━━
1. In AWS Console search bar, type: IAM
2. Click "IAM" in results
3. You're now in Identity and Access Management

STEP 2: Enable MFA for Root Account First
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. In IAM Dashboard, look for "Security recommendations"
2. Click "Add MFA for root user"
3. Click "Activate MFA"
4. Choose MFA device type:
   □ Authenticator app (recommended - Google Authenticator)
   □ Security key
   □ Hardware TOTP token
5. If Authenticator app:
   - Open Google Authenticator on phone
   - Tap "+" → "Scan QR code"
   - Scan QR code shown on screen
   - Enter two consecutive 6-digit codes
   - Click "Add MFA"
6. ✅ Root MFA is now enabled

STEP 3: Create Admin IAM User
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. In left sidebar, click "Users"
2. Click "Create user" button
3. Enter user details:
   - User name: admin-yourname (example: admin-john)
   - ☑️ Check "Provide user access to AWS Management Console"
   - Select "I want to create an IAM user"
   - Console password: Custom password or Auto-generated
   - ☑️ "User must create new password at next sign-in" (optional)
4. Click "Next"

STEP 4: Set Permissions for Admin User
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Select "Attach policies directly"
2. Search for: AdministratorAccess
3. ☑️ Check the box next to "AdministratorAccess"
4. Click "Next"

STEP 5: Add Tags (Optional but Recommended)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Add tags to organize resources:
   Key: Department    Value: IT
   Key: Environment   Value: Production
   Key: Owner         Value: YourName
2. Click "Create user"

STEP 6: Save Access Information
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Note your:
   - Account ID (12-digit number shown in top right)
   - IAM user sign-in URL: 
     https://[account-id].signin.aws.amazon.com/console
2. Click "Return to users list"

STEP 7: Sign Out and Sign Back In as IAM User
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click Account menu → Sign Out
2. Go to IAM user sign-in URL
3. Enter: Account ID, Username, Password
4. ✅ Now you're working as IAM user (not root)
```

### AWS Key Services Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS Core Services Map                        │
│                                                                 │
│  COMPUTE                    STORAGE                            │
│  ┌─────────────────────┐   ┌─────────────────────┐            │
│  │ EC2 - Virtual        │   │ S3 - Object storage  │            │
│  │ machines             │   │ (files, images,      │            │
│  │                      │   │  backups)            │            │
│  │ Lambda - Run code    │   │                      │            │
│  │ without servers      │   │ EBS - Block storage  │            │
│  │                      │   │ (hard drive for EC2) │            │
│  │ ECS - Container      │   │                      │            │
│  │ orchestration        │   │ EFS - Shared file    │            │
│  └─────────────────────┘   │ system               │            │
│                             └─────────────────────┘            │
│  DATABASE                   NETWORKING                         │
│  ┌─────────────────────┐   ┌─────────────────────┐            │
│  │ RDS - Relational DB  │   │ VPC - Private        │            │
│  │ (MySQL, PostgreSQL,  │   │ network in cloud     │            │
│  │  SQL Server)         │   │                      │            │
│  │                      │   │ Route 53 - DNS       │            │
│  │ DynamoDB - NoSQL     │   │ service              │            │
│  │ key-value store      │   │                      │            │
│  │                      │   │ CloudFront - CDN     │            │
│  │ ElastiCache - In-    │   │ content delivery     │            │
│  │ memory caching       │   └─────────────────────┘            │
│  └─────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create Your First EC2 Instance

```
What is EC2?
EC2 = Elastic Compute Cloud
= Virtual computer (server) in the cloud
= You choose: CPU, RAM, storage, OS

STEP 1: Navigate to EC2
━━━━━━━━━━━━━━━━━━━━━━━
1. AWS Console → Search "EC2"
2. Click "EC2" → You're in EC2 Dashboard
3. Click "Launch Instance" (orange button)

STEP 2: Name Your Instance
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Under "Name and tags":
   - Name: my-first-server
   - This is just a label for organization

STEP 3: Choose an AMI (Operating System)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AMI = Amazon Machine Image = pre-configured OS template

1. Under "Application and OS Images (Amazon Machine Image)":
2. "Quick Start" tab shows common options:
   ┌──────────────────────────────────────────────────┐
   │ ● Amazon Linux 2023 (Free tier eligible) ← Best  │
   │   AWS-optimized, lightweight, secure             │
   ├──────────────────────────────────────────────────┤
   │ ○ Ubuntu 22.04 LTS (Free tier eligible)          │
   │   Popular Linux distribution                     │
   ├──────────────────────────────────────────────────┤
   │ ○ Windows Server 2022 (Additional cost)          │
   │   For Windows-based applications                 │
   └──────────────────────────────────────────────────┘
3. Select: Amazon Linux 2023 AMI
4. Architecture: 64-bit (x86) 

STEP 4: Choose Instance Type
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Instance Type = Size of your virtual computer

1. Click "Compare instance types" to understand options
2. Instance naming pattern:
   t3.micro
   │ │  └──── Size (nano, micro, small, medium, large, xlarge)
   │ └─────── Generation (newer = better)
   └───────── Family (t=general, c=compute, r=memory, g=GPU)

3. Free tier: Select t2.micro or t3.micro
   ┌─────────────────────────────────────────┐
   │ t2.micro: 1 vCPU, 1 GB RAM - FREE TIER  │
   │ t2.small: 1 vCPU, 2 GB RAM              │
   │ t2.medium: 2 vCPU, 4 GB RAM             │
   │ t3.micro: 2 vCPU, 1 GB RAM - FREE TIER  │
   │ m5.large: 2 vCPU, 8 GB RAM              │
   └─────────────────────────────────────────┘

STEP 5: Create Key Pair (for SSH access)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Key pair = digital lock/key system to securely access server

1. Under "Key pair (login)":
2. Click "Create new key pair"
3. In popup window:
   - Key pair name: my-server-key
   - Key pair type: ● RSA
   - Private key file format: ● .pem (for Mac/Linux)
                              ● .ppk (for Windows PuTTY)
4. Click "Create key pair"
5. ⬇️ File downloads automatically to your computer
6. ⚠️ SAVE THIS FILE! You cannot download it again!
   Store in safe location: ~/.ssh/my-server-key.pem

STEP 6: Configure Network Settings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Under "Network settings":
2. Click "Edit" to see all options
3. VPC: Keep default VPC
4. Subnet: Keep default (any Availability Zone)
5. Auto-assign public IP: Enable ← So you can access it
6. Firewall (Security Groups):
   - Select "Create security group"
   - Security group name: my-server-sg
   - Description: Security group for my first server
   - Inbound rules:
     ┌─────────────────────────────────────────────────┐
     │ Rule 1 (already added):                         │
     │   Type: SSH | Port: 22 | Source: My IP          │
     │   Purpose: Remote access from your computer     │
     └─────────────────────────────────────────────────┘

STEP 7: Configure Storage
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Under "Configure storage":
2. Default: 8 GiB gp3 (free tier allows up to 30 GiB)
3. Change to 20 GiB for more space
4. Volume type options:
   - gp3: General Purpose SSD (recommended)
   - io1/io2: High performance SSD (more expensive)
   - st1: Throughput optimized HDD (big data)

STEP 8: Review and Launch
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Review "Summary" panel on right:
   ┌─────────────────────────────────────┐
   │ Number of instances: 1              │
   │ AMI: Amazon Linux 2023             │
   │ Instance type: t2.micro             │
   │ Key pair: my-server-key             │
   │ Estimated cost: $0 (free tier)      │
   └─────────────────────────────────────┘
2. Click "Launch instance" (orange button)

STEP 9: Verify Instance is Running
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click "View all instances"
2. See your instance in the list
3. Wait for "Instance State" = running (green dot)
4. Wait for "Status Check" = 2/2 checks passed
   (takes 2-5 minutes)
5. Note your "Public IPv4 address" (example: 54.123.45.67)
```

### Manual Process: Create S3 Bucket

```
What is S3?
S3 = Simple Storage Service
= Store any file: images, videos, backups, code
= Virtually unlimited storage
= 99.999999999% (11 9's) durability

STEP 1: Navigate to S3
━━━━━━━━━━━━━━━━━━━━━━
1. AWS Console → Search "S3"
2. Click "S3"
3. Click "Create bucket"

STEP 2: Configure Bucket
━━━━━━━━━━━━━━━━━━━━━━━━
1. General configuration:
   - AWS Region: US East (N. Virginia) us-east-1
   - Bucket name: myapp-storage-2024-yourinitials
     ⚠️ Must be globally unique across ALL AWS accounts
     ⚠️ Lowercase letters, numbers, hyphens only
     ⚠️ 3-63 characters long

2. Object Ownership:
   ● ACLs disabled (recommended)
     - Bucket owner has full control
     - Simpler security management

3. Block Public Access settings:
   ☑️ Block all public access (recommended for most cases)
   ⚠️ Only UNCHECK if hosting a public website

4. Bucket Versioning:
   ● Disable (for learning)
   ● Enable (for production - keeps all file versions)

5. Default encryption:
   - Encryption type: SSE-S3 (AWS managed keys)
   - Automatically encrypts all stored objects

6. Click "Create bucket"

STEP 3: Upload Your First File
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click on your bucket name
2. Click "Upload"
3. Click "Add files"
4. Select a file from your computer
5. Under "Properties":
   - Storage class: Standard (for frequently accessed data)
   ┌─────────────────────────────────────────────┐
   │ Storage Classes and Use Cases:              │
   │ Standard: Frequent access ($0.023/GB/month) │
   │ IA (Infrequent Access): Occasional access   │
   │ Glacier: Long-term archive (very cheap)     │
   │ Intelligent-Tiering: Auto-moves between     │
   │   tiers based on access patterns            │
   └─────────────────────────────────────────────┘
6. Click "Upload"
7. Wait for "Upload succeeded" message
```

---

## 6. Azure Core Concepts & Manual Setup

### Azure Global Infrastructure

```
Azure Global Infrastructure:
┌─────────────────────────────────────────────────────────────────┐
│                    Azure Global Map                             │
│                                                                 │
│  Regions: 60+ regions worldwide                                 │
│  Availability Zones: Data centers within regions               │
│  Azure CDN: Edge locations globally                             │
│                                                                 │
│  Region Example: East US (Virginia)                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  Region (East US)                       │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │    │
│  │  │    Zone 1   │  │    Zone 2   │  │    Zone 3   │    │    │
│  │  │             │  │             │  │             │    │    │
│  │  │ Data Center │  │ Data Center │  │ Data Center │    │    │
│  │  │ (physically │  │ (physically │  │ (physically │    │    │
│  │  │  separate)  │  │  separate)  │  │  separate)  │    │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Azure Region Pairs:                                            │
│  Each region is paired with another for disaster recovery       │
│  East US ←→ West US                                            │
│  North Europe ←→ West Europe                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Azure Hierarchy (CRITICAL CONCEPT)

```
Azure Resource Organization Hierarchy:
┌─────────────────────────────────────────────────────────────────┐
│                     Management Groups                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │           Your Organization's Root Group                  │  │
│  │  ┌──────────────────┐  ┌────────────────────────────┐    │  │
│  │  │   IT Division    │  │      Business Division      │    │  │
│  │  │ Management Group │  │      Management Group       │    │  │
│  │  └──────────────────┘  └────────────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                       Subscriptions                             │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │   Development    │  │   Production     │                    │
│  │   Subscription   │  │   Subscription   │                    │
│  └──────────────────┘  └──────────────────┘                    │
├─────────────────────────────────────────────────────────────────┤
│                    Resource Groups                              │
│  ┌────────────────────┐  ┌─────────────────────────────────┐   │
│  │  rg-webapp-dev     │  │         rg-webapp-prod          │   │
│  │  (logical container│  │   (all production resources     │   │
│  │   for related      │  │    grouped here)                │   │
│  │   resources)       │  └─────────────────────────────────┘   │
│  └────────────────────┘                                        │
├─────────────────────────────────────────────────────────────────┤
│                       Resources                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │   VM     │ │ Storage  │ │ Database │ │  VNet    │          │
│  │          │ │ Account  │ │          │ │          │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create Azure Free Account

```
STEP 1: Navigate to Azure Free Account
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Open browser → go to: https://azure.microsoft.com/free
2. Click "Start free" button

STEP 2: Microsoft Account Sign-In
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Option A - Have Microsoft/Outlook account:
1. Click "Sign in" 
2. Enter existing Microsoft account email
3. Enter password

Option B - Create new Microsoft account:
1. Click "Create one"
2. Enter new email or use existing email
3. Create password for Microsoft account
4. Verify with code sent to email/phone

STEP 3: Azure Sign-Up Form
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Your profile tab:
   - Country/Region: Select your country
   - First name, Last name
   - Email address
   - Phone: Enter mobile number
   - Click "Text me" or "Call me" for verification code
   - Enter verification code
   - Company: Optional

2. Identity verification by phone:
   - Country code
   - Phone number
   - Click "Text me" 
   - Enter code received

3. Identity verification by card:
   - Card number
   - Expiration date, CVV
   - Name on card, Billing address
   ℹ️ $1 temporary hold (refunded in 3-5 days)
   ℹ️ Won't be charged unless you upgrade to paid

4. Technical support plan:
   ● No technical support (Free) ← Select this
   
5. Agreement:
   ☑️ I agree to the subscription agreement...
   Click "Sign up"

STEP 4: Welcome to Azure!
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click "Go to the portal" or go to:
   https://portal.azure.com
2. You're now in the Azure Portal

STEP 5: Azure Free Account Benefits
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌───────────────────────────────────────────────────┐
│             Azure Free Tier Includes:             │
├───────────────────────────────────────────────────┤
│ $200 credit for first 30 days                     │
│ 12 months free popular services:                  │
│   - 750 hours Linux/Windows VMs (B1s)             │
│   - 5 GB Azure Blob Storage                       │
│   - 250 GB SQL Database                           │
│   - 5 GB Azure Cosmos DB                          │
│ Always free services:                             │
│   - 1 million Azure Functions requests/month      │
│   - 10 web apps (Azure App Service free tier)     │
│   - Azure DevOps (5 users)                        │
└───────────────────────────────────────────────────┘
```

### Understanding Azure Portal Layout

```
Azure Portal Layout:
┌─────────────────────────────────────────────────────────────────┐
│ [≡] Microsoft Azure  [🔍 Search...]           [🔔] [?] [👤]    │
├──────────┬──────────────────────────────────────────────────────┤
│          │                                                       │
│ Favorites│  Home Page                                           │
│ ──────── │  ┌──────────────────────────────────────────────┐   │
│ Home     │  │  Azure services                              │   │
│ Dashboard│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │   │
│          │  │  │  VM  │ │Storage│ │  DB  │ │ More │       │   │
│ All      │  │  └──────┘ └──────┘ └──────┘ └──────┘       │   │
│ resources│  └──────────────────────────────────────────────┘   │
│          │  ┌──────────────────────────────────────────────┐   │
│ Resource │  │  Navigate                                    │   │
│ Groups   │  │  ┌──────────────┐  ┌──────────────────┐     │   │
│          │  │  │Subscriptions │  │ Resource Groups  │     │   │
│ Virtual  │  │  └──────────────┘  └──────────────────┘     │   │
│ Machines │  └──────────────────────────────────────────────┘   │
│          │  ┌──────────────────────────────────────────────┐   │
│ Storage  │  │  Recent Resources                            │   │
│ Accounts │  │  Shows recently accessed resources           │   │
│          │  └──────────────────────────────────────────────┘   │
└──────────┴──────────────────────────────────────────────────────┘
```

### Manual Process: Create Resource Group in Azure

```
Resource Group = Logical container for Azure resources
= Organize related resources together
= Apply policies, permissions, tags to group

STEP 1: Navigate to Resource Groups
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Azure Portal (https://portal.azure.com)
2. Search bar at top: type "Resource groups"
3. Click "Resource groups" in results
4. Click "+ Create" button

STEP 2: Configure Resource Group
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Basics tab:
   - Subscription: Azure subscription 1 (or your subscription)
   - Resource group name: rg-myapp-dev
     Naming convention: rg-[project]-[environment]
   - Region: (US) East US
     ℹ️ Resource group region = where metadata is stored
     ℹ️ Resources inside can be in different regions

2. Tags tab (recommended for organization):
   ┌──────────────────┬──────────────────┐
   │ Name             │ Value            │
   ├──────────────────┼──────────────────┤
   │ Environment      │ Development      │
   │ Project          │ MyApp            │
   │ Owner            │ YourName         │
   │ CostCenter       │ IT-001           │
   └──────────────────┴──────────────────┘

3. Review + Create tab:
   - Review settings summary
   - Click "Create" button

STEP 3: Verify Creation
━━━━━━━━━━━━━━━━━━━━━━━
1. Notification bell (🔔) shows "Resource group created"
2. Click "Go to resource group"
3. You'll see empty resource group ready for resources
```

### Manual Process: Create Azure Virtual Machine

```
STEP 1: Navigate to Virtual Machines
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Azure Portal → Search "Virtual machines"
2. Click "+ Create" → "Azure virtual machine"

STEP 2: Basics Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Project Details:
   - Subscription: Your subscription
   - Resource group: rg-myapp-dev (select existing)
     OR click "Create new" → enter name

2. Instance Details:
   - Virtual machine name: vm-myapp-web-01
     Naming convention: vm-[project]-[role]-[number]
   - Region: (US) East US
   - Availability options: 
     ● No infrastructure redundancy (for learning)
     ○ Availability Zone (for production)
   - Security type: Standard
   - Image: Ubuntu Server 22.04 LTS - x64 Gen2
     (Or Windows Server 2022 if needed)
   - Size: Click "See all sizes"
     ┌──────────────────────────────────────────────┐
     │ Recommended free tier options:               │
     │ B1s: 1 vCPU, 1 GB RAM - FREE TIER           │
     │ B2s: 2 vCPU, 4 GB RAM                       │
     │ D2s_v3: 2 vCPU, 8 GB RAM (general purpose)  │
     └──────────────────────────────────────────────┘
     Select: Standard_B1s (free tier)
     Click "Select"

STEP 3: Administrator Account
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
For Linux VM:
   - Authentication type: ● SSH public key (recommended)
   - Username: azureuser
   - SSH public key source: 
     ● Generate new key pair (easiest for beginners)
   - Key pair name: vm-myapp-key
   
   ⚠️ OR use Password:
   - Authentication type: ● Password
   - Username: azureuser
   - Password: [Strong password - min 12 chars]
   - Confirm password

STEP 4: Inbound Port Rules
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Public inbound ports: ● Allow selected ports
2. Select inbound ports: SSH (22)
   ☑️ SSH (22)  ← Required to connect to Linux VM
   ☑️ HTTP (80) ← For web server access
   ⚠️ Only for learning - use Network Security Groups in production

STEP 5: Disks Tab
━━━━━━━━━━━━━━━━━
1. OS disk type:
   ● Standard SSD (Locally-redundant storage) ← Free tier
   ○ Premium SSD (better performance)
   ○ Standard HDD (cheapest, slowest)
2. Leave other settings as default

STEP 6: Networking Tab
━━━━━━━━━━━━━━━━━━━━━━
1. Virtual network: 
   ● (new) rg-myapp-dev-vnet (auto-created)
2. Subnet: 
   ● (new) default (10.0.0.0/24)
3. Public IP: 
   ● (new) vm-myapp-web-01-ip
4. NIC network security group: Basic
5. Public inbound ports: (same as previous tab)

STEP 7: Management Tab
━━━━━━━━━━━━━━━━━━━━━━
1. Microsoft Defender for Cloud: 
   ● Basic (free) ← for learning
2. Auto-shutdown: 
   ● Enable → Set time: 11:00 PM (save costs!)
   - Time zone: Your timezone

STEP 8: Review + Create
━━━━━━━━━━━━━━━━━━━━━━━
1. Review validation passed (green checkmark)
2. Review estimated cost at bottom
3. Click "Create"
4. If generating SSH key pair: 
   - Popup appears: "Generate new key pair"
   - Click "Download private key and create resource"
   - ⬇️ Saves .pem file to your computer
   - ⚠️ Save this file securely!

STEP 5: Monitor Deployment
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Deployment page shows progress:
   - Creating virtual machine... (2-5 minutes)
   - Creating public IP address...
   - Creating network interface...
   - Creating virtual network...
2. When complete: "Your deployment is complete" ✅
3. Click "Go to resource"
```

### Azure Key Services Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                   Azure Core Services Map                       │
│                                                                 │
│  COMPUTE                    STORAGE                            │
│  ┌─────────────────────┐   ┌─────────────────────┐            │
│  │ Virtual Machines -   │   │ Blob Storage - Object│            │
│  │ IaaS compute         │   │ storage (like AWS S3)│            │
│  │                      │   │                      │            │
│  │ Azure Functions -    │   │ Azure Files - Managed│            │
│  │ Serverless (like     │   │ file share (SMB/NFS) │            │
│  │ AWS Lambda)          │   │                      │            │
│  │                      │   │ Azure Disk - Block   │            │
│  │ App Service - PaaS   │   │ storage (like EBS)   │            │
│  │ web apps & APIs      │   └─────────────────────┘            │
│  └─────────────────────┘                                       │
│  DATABASE                   NETWORKING                         │
│  ┌─────────────────────┐   ┌─────────────────────┐            │
│  │ Azure SQL Database   │   │ Virtual Network      │            │
│  │ (managed SQL Server) │   │ (VNet) - like AWS    │            │
│  │                      │   │ VPC                  │            │
│  │ Cosmos DB - Global   │   │                      │            │
│  │ NoSQL database       │   │ Azure DNS - Domain   │            │
│  │                      │   │ name service         │            │
│  │ Azure Database for   │   │                      │            │
│  │ PostgreSQL, MySQL    │   │ Azure CDN - Content  │            │
│  └─────────────────────┘   │ delivery network     │            │
│                             └─────────────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create Azure Storage Account & Blob Container

```
STEP 1: Navigate to Storage Accounts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Azure Portal → Search "Storage accounts"
2. Click "+ Create"

STEP 2: Basic Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Project details:
   - Subscription: Your subscription
   - Resource group: rg-myapp-dev

2. Instance details:
   - Storage account name: mystorageaccount2024
     ⚠️ Must be globally unique (lowercase, numbers only)
     ⚠️ 3-24 characters
   - Region: East US
   - Performance:
     ● Standard (HDD-based, most use cases)
     ○ Premium (SSD-based, high performance)
   - Redundancy:
     ┌──────────────────────────────────────────────────┐
     │ LRS - Locally Redundant Storage                  │
     │   3 copies in ONE data center (cheapest)         │
     │   Protects against: disk failure                 │
     ├──────────────────────────────────────────────────┤
     │ ZRS - Zone Redundant Storage                     │
     │   3 copies across 3 Availability Zones           │
     │   Protects against: data center failure          │
     ├──────────────────────────────────────────────────┤
     │ GRS - Geo-Redundant Storage                      │
     │   6 copies: 3 in primary, 3 in secondary region  │
     │   Protects against: regional disaster            │
     ├──────────────────────────────────────────────────┤
     │ GZRS - Geo-Zone Redundant Storage                │
     │   Most protection (most expensive)               │
     └──────────────────────────────────────────────────┘
     Select: LRS for learning

STEP 3: Advanced Tab (Optional)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Require secure transfer (HTTPS): ● Enabled ✅
2. Enable blob public access: ○ Disabled (secure default)
3. Minimum TLS version: TLS 1.2 ✅

STEP 4: Review + Create
━━━━━━━━━━━━━━━━━━━━━━━
1. Click "Review"
2. Review settings
3. Click "Create"
4. Wait for deployment (30-60 seconds)
5. Click "Go to resource"

STEP 5: Create Blob Container
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. In Storage Account → Left sidebar: "Containers"
2. Click "+ Container"
3. In popup:
   - Name: images (lowercase, no spaces)
   - Public access level:
     ● Private (no anonymous access) ← Recommended
     ○ Blob (anonymous read for blobs)
     ○ Container (anonymous read for container)
4. Click "Create"

STEP 6: Upload a Blob (File)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click on "images" container
2. Click "Upload"
3. In upload panel:
   - Click folder icon → select file from computer
   - Advanced options:
     - Authentication type: Account key
     - Blob type: Block blob (for files)
     - Block size: 8 MiB (default)
4. Click "Upload"
5. File appears in container list ✅
```

---

## 7. AWS vs Azure Architecture Comparison

### Service Name Mapping

```
┌─────────────────────────────────────────────────────────────────┐
│                  AWS vs Azure Service Comparison                │
├──────────────────────┬──────────────────────┬───────────────────┤
│   Category           │      AWS             │      Azure        │
├──────────────────────┼──────────────────────┼───────────────────┤
│ COMPUTE              │                      │                   │
│ Virtual Machines     │ EC2                  │ Virtual Machines  │
│ Containers           │ ECS / EKS            │ AKS / ACI         │
│ Serverless           │ Lambda               │ Azure Functions   │
│ PaaS Web Apps        │ Elastic Beanstalk    │ App Service       │
│ Batch Processing     │ AWS Batch            │ Azure Batch       │
├──────────────────────┼──────────────────────┼───────────────────┤
│ STORAGE              │                      │                   │
│ Object Storage       │ S3                   │ Blob Storage      │
│ Block Storage        │ EBS                  │ Azure Disks       │
│ File Storage         │ EFS                  │ Azure Files       │
│ Archive Storage      │ S3 Glacier           │ Archive Storage   │
├──────────────────────┼──────────────────────┼───────────────────┤
│ DATABASE             │                      │                   │
│ Relational DB        │ RDS                  │ Azure SQL         │
│ NoSQL                │ DynamoDB             │ Cosmos DB         │
│ In-Memory Cache      │ ElastiCache          │ Azure Cache Redis │
│ Data Warehouse       │ Redshift             │ Synapse Analytics │
├──────────────────────┼──────────────────────┼───────────────────┤
│ NETWORKING           │                      │                   │
│ Virtual Network      │ VPC                  │ Virtual Network   │
│ Load Balancing       │ ELB/ALB/NLB          │ Azure Load Bal.   │
│ DNS                  │ Route 53             │ Azure DNS         │
│ CDN                  │ CloudFront           │ Azure CDN         │
│ VPN                  │ VPN Gateway          │ VPN Gateway       │
│ Private Connection   │ Direct Connect       │ ExpressRoute      │
├──────────────────────┼──────────────────────┼───────────────────┤
│ SECURITY & IDENTITY  │                      │                   │
│ Identity             │ IAM                  │ Azure AD (Entra)  │
│ Key Management       │ KMS                  │ Key Vault         │
│ Security Center      │ Security Hub         │ Defender for Cloud│
│ DDoS Protection      │ Shield               │ Azure DDoS        │
├──────────────────────┼──────────────────────┼───────────────────┤
│ MONITORING           │                      │                   │
│ Monitoring           │ CloudWatch           │ Azure Monitor     │
│ Logging              │ CloudTrail           │ Activity Log      │
│ Infrastructure as    │ CloudFormation       │ ARM Templates     │
│ Code                 │                      │ / Bicep           │
│ Cost Management      │ Cost Explorer        │ Cost Management   │
└──────────────────────┴──────────────────────┴───────────────────┘
```

### Key Architectural Differences

```
AWS Architecture Philosophy:
┌─────────────────────────────────────────────────────────────────┐
│  AWS = Service-First Approach                                   │
│                                                                 │
│  Many specialized, independent services                         │
│  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                 │
│  │ EC2 │  │ S3  │  │ RDS │  │ VPC │  │ IAM │                 │
│  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘                 │
│     │        │         │         │         │                    │
│     └────────┴─────────┴─────────┴─────────┘                   │
│              You connect services together                      │
│                                                                 │
│  AWS Regions contain Availability Zones                         │
│  Resources deployed to specific AZ or Region level             │
│  Global services: IAM, Route 53, CloudFront                    │
└─────────────────────────────────────────────────────────────────┘

Azure Architecture Philosophy:
┌─────────────────────────────────────────────────────────────────┐
│  Azure = Enterprise-Integration Approach                        │
│                                                                 │
│  Resource Groups = Core organizational unit                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │            Subscription                                 │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │             Resource Group                       │   │    │
│  │  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐        │   │    │
│  │  │  │  VM  │  │ VNet │  │  DB  │  │ Disk │        │   │    │
│  │  │  └──────┘  └──────┘  └──────┘  └──────┘        │   │    │
│  │  │  All resources managed as one unit               │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Deep integration with Microsoft 365, Active Directory          │
│  Azure = Best for Microsoft-centric organizations               │
└─────────────────────────────────────────────────────────────────┘
```

### Pricing Model Differences

```
AWS Pricing:
┌─────────────────────────────────────────────────────────────────┐
│ On-Demand: Pay by the second/hour, no commitment                │
│ Reserved Instances: 1 or 3 year commitment (up to 75% off)     │
│ Spot Instances: Bid for unused capacity (up to 90% off)         │
│ Savings Plans: Flexible commitment based on $/hour              │
│                                                                 │
│ Example EC2 Pricing (t3.medium, us-east-1):                    │
│   On-Demand:     $0.0416/hour = ~$30/month                     │
│   Reserved (1yr): ~$0.026/hour = ~$19/month (37% savings)      │
│   Reserved (3yr): ~$0.016/hour = ~$12/month (61% savings)      │
│   Spot:          ~$0.013/hour = ~$9/month (68% savings)        │
└─────────────────────────────────────────────────────────────────┘

Azure Pricing:
┌─────────────────────────────────────────────────────────────────┐
│ Pay-As-You-Go: Pay by the second, no commitment                 │
│ Reserved VM Instances: 1 or 3 year (up to 72% savings)        │
│ Spot VMs: Unused capacity (up to 90% off)                      │
│ Azure Hybrid Benefit: Use existing Windows/SQL licenses         │
│   (Extra savings for Microsoft customers!)                      │
│                                                                 │
│ Example VM Pricing (B2s, East US):                             │
│   Pay-As-You-Go:   $0.0496/hour = ~$36/month                   │
│   Reserved (1yr):  ~$0.030/hour = ~$22/month                   │
│   Hybrid Benefit:  Additional 40% off with Windows licenses    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Networking Fundamentals

### Virtual Private Cloud/Network Concepts

```
VPC (AWS) / VNet (Azure) Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                    Your Virtual Network                         │
│                  CIDR: 10.0.0.0/16                             │
│                  (65,536 IP addresses)                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │               Public Subnet                             │    │
│  │               10.0.1.0/24 (256 IPs)                    │    │
│  │  ┌──────────┐  ┌──────────┐                            │    │
│  │  │  Web     │  │  Load    │                            │    │
│  │  │ Server   │  │ Balancer │                            │    │
│  │  │(Public IP│  │(Public IP│                            │    │
│  │  │ assigned)│  │ assigned)│                            │    │
│  │  └──────────┘  └──────────┘                            │    │
│  │  [Internet Gateway attached - internet accessible]      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                     │
│                    [Route Table]                                │
│                           │                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │               Private Subnet                            │    │
│  │               10.0.2.0/24 (256 IPs)                    │    │
│  │  ┌──────────┐  ┌──────────┐                            │    │
│  │  │  App     │  │ Database │                            │    │
│  │  │ Server   │  │ Server   │                            │    │
│  │  │(Private  │  │(Private  │                            │    │
│  │  │ IP only) │  │ IP only) │                            │    │
│  │  └──────────┘  └──────────┘                            │    │
│  │  [No direct internet access - more secure]              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                     │
│                      [NAT Gateway]                             │
│                    (outbound only)                              │
│                           │                                     │
│               [INTERNET GATEWAY / VPN]                         │
│                           │                                     │
│                       INTERNET                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Security Groups vs Network Security Groups

```
AWS Security Groups:
┌─────────────────────────────────────────────────────────────────┐
│ Security Group = Virtual firewall for EC2 instances             │
│                                                                 │
│ Characteristics:                                                │
│ ✅ Stateful (return traffic automatically allowed)              │
│ ✅ Applied at instance level                                    │
│ ✅ Allow rules only (no deny rules)                            │
│ ✅ All rules evaluated together                                │
│                                                                 │
│ Example Security Group for Web Server:                         │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ Inbound Rules:                                             │  │
│ │ Type    Protocol  Port  Source                            │  │
│ │ HTTP    TCP       80    0.0.0.0/0 (anyone)               │  │
│ │ HTTPS   TCP       443   0.0.0.0/0 (anyone)               │  │
│ │ SSH     TCP       22    203.0.113.0/32 (your IP only)    │  │
│ │                                                            │  │
│ │ Outbound Rules:                                            │  │
│ │ All traffic  All  All   0.0.0.0/0 (default - allow all)  │  │
│ └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

Azure Network Security Groups (NSG):
┌─────────────────────────────────────────────────────────────────┐
│ NSG = Azure's firewall for VNets and subnets                   │
│                                                                 │
│ Characteristics:                                                │
│ ✅ Stateful (like AWS Security Groups)                         │
│ ✅ Applied to subnet OR network interface                      │
│ ✅ Both Allow and Deny rules                                   │
│ ✅ Priority-based (lower number = higher priority)             │
│                                                                 │
│ Example NSG Rules:                                              │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ Inbound Security Rules:                                    │  │
│ │ Priority Name      Port  Protocol Source    Action         │  │
│ │ 100     AllowHTTP  80    TCP      *         Allow          │  │
│ │ 110     AllowHTTPS 443   TCP      *         Allow          │  │
│ │ 120     AllowSSH   22    TCP      My IP     Allow          │  │
│ │ 65000   AllowVNet  *     *        VirtualNet Allow         │  │
│ │ 65001   AllowLB    *     *        AzureLoadB Allow         │  │
│ │ 65500   DenyAll    *     *        *          Deny           │  │
│ └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create VPC in AWS

```
STEP 1: Navigate to VPC
━━━━━━━━━━━━━━━━━━━━━━━
1. AWS Console → Search "VPC"
2. Click "VPC" → "Your VPCs"
3. Click "Create VPC"

STEP 2: VPC Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Resources to create:
   ● VPC and more (creates subnets, route tables, gateways)
   ○ VPC only (manual subnet creation)
   Select: ● VPC and more

2. Name tag auto-generation: myapp
   (Creates: myapp-vpc, myapp-subnet-public1-us-east-1a, etc.)

3. IPv4 CIDR block:
   10.0.0.0/16
   ┌────────────────────────────────────────────────┐
   │ CIDR Notation Explained:                       │
   │ 10.0.0.0/16 = 65,536 IP addresses available   │
   │ 10.0.0.0/24 = 256 IP addresses available      │
   │ 10.0.0.0/28 = 16 IP addresses available       │
   │                                                │
   │ Common private IP ranges:                      │
   │ 10.0.0.0/8    (10.x.x.x)                      │
   │ 172.16.0.0/12 (172.16.x.x - 172.31.x.x)      │
   │ 192.168.0.0/16 (192.168.x.x)                  │
   └────────────────────────────────────────────────┘

4. Number of Availability Zones: 2
5. Number of public subnets: 2
6. Number of private subnets: 2

7. NAT gateways: None (costs money - for learning)
   In production: Select "1 per AZ" or "In 1 AZ"

8. VPC endpoints: None

9. DNS options:
   ☑️ Enable DNS hostnames
   ☑️ Enable DNS resolution

STEP 3: Review and Create
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Review the "Preview" diagram showing:
   - VPC with CIDR
   - Public and Private subnets
   - Internet Gateway
   - Route tables
2. Click "Create VPC"
3. Wait for all resources to be created (~30 seconds)
4. Click "View VPC" when complete
```

---

## 9. Storage Concepts

### Storage Types Explained

```
┌─────────────────────────────────────────────────────────────────┐
│                    Three Types of Cloud Storage                 │
│                                                                 │
│  1. BLOCK STORAGE (like a hard drive)                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  [   Block 1   ][   Block 2   ][   Block 3   ][Block 4]  │  │
│  │  OS sees it as a raw disk, can format with any filesystem  │  │
│  │  Very fast, low latency                                   │  │
│  │  AWS: EBS (Elastic Block Store)                           │  │
│  │  Azure: Azure Managed Disks                               │  │
│  │  Use case: Database storage, OS drive                     │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  2. FILE STORAGE (like a network drive)                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  /home/user/                                              │  │
│  │  ├── documents/                                           │  │
│  │  │   ├── report.pdf                                       │  │
│  │  │   └── data.xlsx                                        │  │
│  │  └── images/                                              │  │
│  │      └── photo.jpg                                        │  │
│  │  Traditional file hierarchy, multiple servers can access  │  │
│  │  AWS: EFS (Elastic File System)                           │  │
│  │  Azure: Azure Files                                       │  │
│  │  Use case: Shared content, CMS, home directories         │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  3. OBJECT STORAGE (like a giant key-value store)              │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  bucket/                                                  │  │
│  │  ├── images/photo1.jpg    [metadata: size, type, date]   │  │
│  │  ├── videos/intro.mp4     [metadata: size, type, date]   │  │
│  │  └── backups/db-2024.zip  [metadata: size, type, date]   │  │
│  │  Flat structure, access via URL/API                       │  │
│  │  Massive scale, cheapest, globally accessible            │  │
│  │  AWS: S3                                                  │  │
│  │  Azure: Blob Storage                                      │  │
│  │  Use case: Images, videos, backups, static websites       │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Storage Durability & Redundancy

```
AWS S3 Storage Classes:
┌─────────────────────────────────────────────────────────────────┐
│ Class                  │ Durability │ Availability │ Use Case   │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Standard               │ 11 nines   │ 99.99%       │ Frequent   │
│                        │ 99.999...% │              │ access     │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Standard-IA            │ 11 nines   │ 99.9%        │ Monthly    │
│ (Infrequent Access)    │            │              │ access     │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Intelligent-Tiering    │ 11 nines   │ 99.9%        │ Unknown    │
│                        │            │              │ access     │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Glacier Instant        │ 11 nines   │ 99.9%        │ Quarterly  │
│ Retrieval              │            │              │ access     │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Glacier Flexible       │ 11 nines   │ 99.99%       │ Archive    │
│ Retrieval              │            │ (after        │ 1-5 min   │
│                        │            │  restore)     │ retrieval  │
├────────────────────────┼────────────┼──────────────┼────────────┤
│ Glacier Deep Archive   │ 11 nines   │ 99.99%       │ Long-term  │
│                        │            │ (after        │ 12hr      │
│                        │            │  restore)     │ retrieval  │
└────────────────────────┴────────────┴──────────────┴────────────┘
```

---

## 10. Compute Resources

### Understanding VM Instance Types

```
AWS EC2 Instance Family Guide:
┌─────────────────────────────────────────────────────────────────┐
│ Family │ Optimized For        │ Example Use Cases              │
├────────┼──────────────────────┼────────────────────────────────┤
│ T      │ General purpose      │ Dev/test, small websites,      │
│        │ (burstable)          │ microservices                  │
├────────┼──────────────────────┼────────────────────────────────┤
│ M      │ General purpose      │ App servers, gaming servers,   │
│        │ (balanced)           │ small databases                │
├────────┼──────────────────────┼────────────────────────────────┤
│ C      │ Compute optimized    │ High-performance computing,    │
│        │ (more CPU)           │ batch processing, game servers │
├────────┼──────────────────────┼────────────────────────────────┤
│ R      │ Memory optimized     │ Big databases, in-memory       │
│        │ (more RAM)           │ analytics, real-time caching   │
├────────┼──────────────────────┼────────────────────────────────┤
│ G      │ GPU instances        │ Machine learning, video        │
│        │ (graphics)           │ encoding, 3D rendering         │
├────────┼──────────────────────┼────────────────────────────────┤
│ I      │ Storage optimized    │ NoSQL databases, data          │
│        │ (fast NVMe SSD)      │ warehousing                    │
├────────┼──────────────────────┼────────────────────────────────┤
│ P      │ ML/AI optimized      │ Machine learning training,     │
│        │ (NVIDIA GPUs)        │ deep learning                  │
└────────┴──────────────────────┴────────────────────────────────┘

Instance Size Guide:
nano < micro < small < medium < large < xlarge < 2xlarge < ...48xlarge

t3.nano:   0.5 GB RAM, 2 vCPU
t3.micro:  1   GB RAM, 2 vCPU  ← Free tier
t3.small:  2   GB RAM, 2 vCPU
t3.medium: 4   GB RAM, 2 vCPU
t3.large:  8   GB RAM, 2 vCPU
t3.xlarge: 16  GB RAM, 4 vCPU
```

### Serverless Computing Concept

```
Traditional (EC2/VM) vs Serverless (Lambda/Functions):

Traditional Approach:
┌─────────────────────────────────────────────────────────────────┐
│  Server always running:                                         │
│                                                                 │
│  Time: 12:00──────────────────────────────────────────23:59    │
│  Cost: ████████████████████████████████████████████████████    │
│  Usage:          ██      ████        ███                        │
│                                                                 │
│  Pay for 24 hours even when barely used                        │
└─────────────────────────────────────────────────────────────────┘

Serverless Approach:
┌─────────────────────────────────────────────────────────────────┐
│  Function runs only when needed:                                │
│                                                                 │
│  Time: 12:00──────────────────────────────────────────23:59    │
│  Cost:           ██      ████        ███                        │
│  Usage:          ██      ████        ███                        │
│                                                                 │
│  Pay ONLY for the milliseconds your code actually runs         │
│  AWS Lambda: $0.0000002 per request                            │
│  First 1 million requests FREE every month                     │
└─────────────────────────────────────────────────────────────────┘

Serverless Use Cases:
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Image processing (resize when uploaded to S3)               │
│  ✅ API backends (handle HTTP requests)                         │
│  ✅ Scheduled tasks (run at specific times)                     │
│  ✅ Event processing (react to database changes)               │
│  ✅ Chatbots and automation                                     │
│                                                                 │
│  ❌ Long-running processes (max 15 min for Lambda)             │
│  ❌ Heavy computation requiring large resources                 │
│  ❌ Applications needing persistent connections                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Security & Identity Management

### AWS IAM Deep Dive

```
IAM Components:
┌─────────────────────────────────────────────────────────────────┐
│                      IAM Structure                              │
│                                                                 │
│  ROOT ACCOUNT                                                   │
│  └── Full access to everything - protect with MFA!             │
│                                                                 │
│  IAM USERS                                                      │
│  ├── john@company.com (admin)                                   │
│  ├── jane@company.com (developer)                               │
│  └── app-service-user (application)                            │
│                                                                 │
│  IAM GROUPS                                                     │
│  ├── Administrators Group → AdministratorAccess policy          │
│  ├── Developers Group → DeveloperAccess policy                  │
│  └── ReadOnly Group → ReadOnlyAccess policy                    │
│                                                                 │
│  IAM ROLES (for services and cross-account)                    │
│  ├── EC2-S3-Access-Role                                        │
│  │   └── EC2 instances can read/write to specific S3 bucket    │
│  ├── Lambda-DynamoDB-Role                                       │
│  │   └── Lambda functions can access DynamoDB                  │
│  └── CrossAccount-Role                                         │
│      └── Users from Account B can access Account A             │
│                                                                 │
│  IAM POLICIES (JSON documents defining permissions)            │
│  ├── AWS Managed Policies (pre-built by Amazon)                │
│  │   ├── AdministratorAccess                                   │
│  │   ├── ReadOnlyAccess                                        │
│  │   └── AmazonS3FullAccess                                   │
│  └── Customer Managed Policies (you create)                    │
│      └── Custom permissions for your specific needs            │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create IAM Policy in AWS

```
STEP 1: Navigate to IAM Policies
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. AWS Console → IAM → Policies (left sidebar)
2. Click "Create policy"

STEP 2: Create Policy Using Visual Editor
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Method A: Visual Editor
1. Click "Add a permission"
2. Service: Search and select "S3"
3. Actions allowed:
   - Filter actions: Check boxes:
     ☑️ ListBucket (see what's in bucket)
     ☑️ GetObject (download files)
     ☑️ PutObject (upload files)
     Leave others unchecked
4. Resources: Choose specific bucket
   - bucket: Add ARN
     Bucket name: my-specific-bucket
   - object: Add ARN  
     Bucket name: my-specific-bucket
     Object name: * (all objects)
5. Click "Next"

Method B: JSON Editor (copy-paste this example)
Click "JSON" tab:
┌─────────────────────────────────────────────────────────────────┐
│ {                                                               │
│   "Version": "2012-10-17",                                      │
│   "Statement": [                                                │
│     {                                                           │
│       "Effect": "Allow",                                        │
│       "Action": [                                               │
│         "s3:ListBucket"                                         │
│       ],                                                        │
│       "Resource": "arn:aws:s3:::my-specific-bucket"            │
│     },                                                          │
│     {                                                           │
│       "Effect": "Allow",                                        │
│       "Action": [                                               │
│         "s3:GetObject",                                         │
│         "s3:PutObject",                                         │
│         "s3:DeleteObject"                                       │
│       ],                                                        │
│       "Resource": "arn:aws:s3:::my-specific-bucket/*"          │
│     }                                                           │
│   ]                                                             │
│ }                                                               │
└─────────────────────────────────────────────────────────────────┘

STEP 3: Policy Details
━━━━━━━━━━━━━━━━━━━━━━
1. Policy name: S3-MyBucket-ReadWrite
2. Description: Allow read and write access to my-specific-bucket
3. Click "Create policy"

STEP 4: Attach Policy to User/Group/Role
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. IAM → Users → Select user
2. Click "Add permissions"
3. Click "Attach policies directly"
4. Search: S3-MyBucket-ReadWrite
5. ☑️ Check the policy
6. Click "Next" → "Add permissions"
```

### Azure Active Directory (Azure AD / Microsoft Entra ID)

```
Azure Identity Structure:
┌─────────────────────────────────────────────────────────────────┐
│              Microsoft Entra ID (Azure AD)                      │
│                                                                 │
│  TENANT = Your organization's identity directory                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │           company.onmicrosoft.com                       │    │
│  │                                                         │    │
│  │  USERS                      GROUPS                      │    │
│  │  ├── admin@company.com      ├── IT Admins               │    │
│  │  ├── dev@company.com        ├── Developers              │    │
│  │  └── user@company.com       └── ReadOnly Users          │    │
│  │                                                         │    │
│  │  SERVICE PRINCIPALS (App identities)                   │    │
│  │  ├── my-app-service-principal                           │    │
│  │  └── automation-script-sp                              │    │
│  │                                                         │    │
│  │  MANAGED IDENTITIES (Azure resource identities)        │    │
│  │  ├── vm-managed-identity (VM accessing Key Vault)       │    │
│  │  └── function-managed-identity                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  RBAC (Role-Based Access Control) - Applied to:                │
│  ├── Management Group level                                     │
│  ├── Subscription level                                         │
│  ├── Resource Group level                                       │
│  └── Individual Resource level                                  │
│                                                                 │
│  Built-in Roles:                                                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Owner       = Full access + manage access                │   │
│  │ Contributor = Full access to manage resources            │   │
│  │ Reader      = View resources only                        │   │
│  │ User Access Administrator = Manage user access           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Assign RBAC Role in Azure

```
STEP 1: Navigate to Resource Group
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Azure Portal → Resource groups
2. Click on your resource group: rg-myapp-dev

STEP 2: Access Control (IAM)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. In left sidebar, click "Access control (IAM)"
2. You see current role assignments
3. Click "+ Add" → "Add role assignment"

STEP 3: Select Role
━━━━━━━━━━━━━━━━━━━
1. "Role" tab shows:
   ┌────────────────────────────────────────────────────────┐
   │ Job function roles:                                    │
   │   Owner                                                │
   │   Contributor      ← Good for developers              │
   │   Reader           ← Good for auditors                │
   │   ...and hundreds more                                 │
   │                                                        │
   │ Privileged administrator roles:                        │
   │   Role Based Access Control Administrator              │
   │   User Access Administrator                            │
   └────────────────────────────────────────────────────────┘
2. Select "Contributor"
3. Click "Next"

STEP 4: Select Member (Who Gets the Role)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Assign access to: 
   ● User, group, or service principal
2. Click "+ Select members"
3. Search for user by name or email
4. Click on user to select them
5. Click "Select"
6. Click "Next"

STEP 5: Review + Assign
━━━━━━━━━━━━━━━━━━━━━━━
1. Review:
   - Role: Contributor
   - Scope: /subscriptions/.../resourceGroups/rg-myapp-dev
   - Member: user@company.com
2. Click "Review + assign"
3. Click "Review + assign" again to confirm
4. ✅ Role assigned successfully
```

---

## 12. Monitoring & Management

### AWS CloudWatch

```
CloudWatch Overview:
┌─────────────────────────────────────────────────────────────────┐
│                      AWS CloudWatch                             │
│                                                                 │
│  METRICS - Numerical data over time                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ EC2 Metrics (collected every 5 min free, 1 min paid):   │    │
│  │  - CPUUtilization (0-100%)                              │    │
│  │  - NetworkIn / NetworkOut (bytes)                       │    │
│  │  - DiskReadOps / DiskWriteOps                          │    │
│  │  - StatusCheckFailed                                    │    │
│  │                                                         │    │
│  │ Custom Metrics: You can push YOUR app metrics           │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ALARMS - Alert when metric crosses threshold                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Example Alarm:                                          │    │
│  │  Metric: CPUUtilization                                 │    │
│  │  Threshold: > 80% for 5 consecutive minutes             │    │
│  │  Action: Send email notification                        │    │
│  │       OR Auto-scale (add more instances)               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  LOGS - Application and system logs                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Log Groups → Log Streams → Log Events                   │    │
│  │                                                         │    │
│  │ /aws/lambda/my-function                                 │    │
│  │   └── 2024/01/15/[LATEST]abc123...                     │    │
│  │         └── 2024-01-15 10:30:15 Function started       │    │
│  │         └── 2024-01-15 10:30:16 Processing data...     │    │
│  │         └── 2024-01-15 10:30:17 Function completed     │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Manual Process: Create CloudWatch Alarm (AWS)

```
STEP 1: Navigate to CloudWatch
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. AWS Console → Search "CloudWatch"
2. Click "CloudWatch"
3. Left sidebar → "Alarms" → "All alarms"
4. Click "Create alarm"

STEP 2: Select Metric
━━━━━━━━━━━━━━━━━━━━━
1. Click "Select metric"
2. Browse or search metrics:
   - Click "EC2" → "Per-Instance Metrics"
   - Find your instance ID
   - Click "CPUUtilization"
3. Click "Select metric"

STEP 3: Specify Metric and Conditions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Metric name: CPUUtilization
2. Statistic: Average
3. Period: 5 minutes

4. Conditions:
   - Threshold type: ● Static
   - Whenever CPUUtilization is: 
     ● Greater than
   - than: 80
   
   ┌────────────────────────────────────────────────────────┐
   │ This means: Alert when CPU averages > 80%              │
   │ for one 5-minute period                                │
   └────────────────────────────────────────────────────────┘
5. Click "Next"

STEP 4: Configure Actions
━━━━━━━━━━━━━━━━━━━━━━━━━
1. Alarm state trigger: ● In alarm
2. Send notification to: 
   - Select an SNS topic OR
   - "Create new topic"
     - Topic name: high-cpu-alerts
     - Email endpoints: your@email.com
     - Click "Create topic"
3. Click "Next"

STEP 5: Name and Description
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Alarm name: High-CPU-Warning-MyServer
2. Alarm description: Alert when EC2 CPU > 80%
3. Click "Next" → "Create alarm"

STEP 6: Confirm Email Subscription
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Check your email
2. Email from AWS: "AWS Notification - Subscription Confirmation"
3. Click "Confirm subscription" link
4. ✅ You'll now receive email when CPU exceeds 80%
```

### Manual Process: Create Alert in Azure Monitor

```
STEP 1: Navigate to Azure Monitor
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Azure Portal → Search "Monitor"
2. Click "Monitor"
3. Left sidebar → "Alerts"
4. Click "+ Create" → "Alert rule"

STEP 2: Select Scope (Resource to Monitor)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click "+ Select scope"
2. Filter by:
   - Resource type: Virtual machines
3. Select your VM from the list
4. Click "Done"

STEP 3: Configure Condition (What to Monitor)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click "+ Add condition"
2. Search signals: "Percentage CPU"
3. Click "Percentage CPU"
4. Configure signal logic:
   - Threshold: Static
   - Aggregation type: Average
   - Operator: Greater than
   - Threshold value: 80
   - Check every: 5 minutes
   - Lookback period: 5 minutes
5. Click "Add"

STEP 4: Actions (What to Do When Alert Fires)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Click "+ Create action group"
2. Action group basics:
   - Subscription: Your subscription
   - Resource group: rg-myapp-dev
   - Action group name: email-alerts-group
   - Display name: EmailAlerts

3. Notifications:
   - Notification type: Email/SMS message/Push/Voice
   - Name: Email-Notification
   - Click edit (pencil icon):
     ☑️ Email: your@email.com
   - Click "OK"

4. Click "Review + create" → "Create"

STEP 5: Alert Rule Details
━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Alert rule name: High-CPU-Alert-MyVM
2. Description: Alert when VM CPU > 80%
3. Severity: 
   - 0 = Critical
   - 1 = Error
   - 2 = Warning ← Select this
   - 3 = Informational
4. Region: East US
5. Click "Review + create" → "Create"
```

---

## 13. Real-World Architecture Patterns

### Three-Tier Web Application Architecture

```
AWS Three-Tier Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    Route 53 (DNS)                               │
│              www.myapp.com → Load Balancer IP                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                 PRESENTATION TIER                               │
│            Application Load Balancer (ALB)                      │
│         Distributes traffic across multiple servers             │
│              ┌──────────────────────┐                          │
│              │   Security Group:    │                          │
│              │   Allow: 80, 443     │                          │
│              └──────────────────────┘                          │
└───────────────┬──────────────────┬─────────────────────────────┘
                │                  │
┌───────────────▼──┐  ┌────────────▼──────────────────────────────┐
│   LOGIC TIER      │  │                                          │
│  Public Subnet A  │  │         Public Subnet B                  │
│   (us-east-1a)   │  │          (us-east-1b)                    │
│  ┌─────────────┐  │  │  ┌─────────────┐                        │
│  │  EC2 Web    │  │  │  │  EC2 Web    │                        │
│  │  Server 1   │  │  │  │  Server 2   │                        │
│  │  (t3.medium)│  │  │  │  (t3.medium)│                        │
│  └──────┬──────┘  │  │  └──────┬──────┘                        │
│         │         │  │         │                               │
└─────────┼─────────┘  └─────────┼───────────────────────────────┘
          │                      │
┌─────────▼──────────────────────▼───────────────────────────────┐
│                       DATA TIER                                 │
│                  Private Subnet (no public access)              │
│                                                                 │
│  ┌──────────────────────┐    ┌──────────────────────────────┐  │
│  │    RDS MySQL         │    │     ElastiCache Redis        │  │
│  │    Primary Instance  │    │     (Session/Query Cache)    │  │
│  │    (db.t3.medium)    │    │     (cache.t3.micro)         │  │
│  └──────────────────────┘    └──────────────────────────────┘  │
│  ┌──────────────────────┐                                       │
│  │    RDS MySQL         │                                       │
│  │    Read Replica      │ ← Read-heavy traffic goes here       │
│  │    (db.t3.medium)    │                                       │
│  └──────────────────────┘                                       │
│         Security Group: Only allow from Web Servers             │
└─────────────────────────────────────────────────────────────────┘
```

### Azure Equivalent Architecture

```
Azure Three-Tier Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    Azure DNS                                    │
│              www.myapp.com → Load Balancer IP                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│               Azure Application Gateway                         │
│         (Load Balancer + WAF Web Application Firewall)          │
└───────────────┬──────────────────┬─────────────────────────────┘
                │                  │
┌───────────────▼──────────────────▼─────────────────────────────┐
│                   Virtual Network (VNet)                        │
│              Address Space: 10.0.0.0/16                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Frontend Subnet 10.0.1.0/24                │    │
│  │  ┌──────────────┐         ┌──────────────┐              │    │
│  │  │  VM Scale    │         │  VM Scale    │              │    │
│  │  │  Set Zone 1  │         │  Set Zone 2  │              │    │
│  │  │  (Web Apps)  │         │  (Web Apps)  │              │    │
│  │  └──────────────┘         └──────────────┘              │    │
│  │  NSG: Allow 80, 443 from AppGateway only                │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Backend Subnet 10.0.2.0/24                 │    │
│  │  ┌──────────────────────┐  ┌─────────────────────────┐  │    │
│  │  │   Azure SQL Database │  │  Azure Cache for Redis  │  │    │
│  │  │   (with Read Replica)│  │  (Session caching)      │  │    │
│  │  └──────────────────────┘  └─────────────────────────┘  │    │
│  │  NSG: Allow only from Frontend Subnet                   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Cost Estimation Guide

```
Monthly Cost Estimation - Small Web Application:

AWS Estimate:
┌─────────────────────────────────────────────────────────────────┐
│ Service              │ Specs            │ Monthly Cost           │
├──────────────────────┼──────────────────┼────────────────────────┤
│ EC2 (2 instances)    │ t3.medium        │ $60  ($30 each)        │
│ Application LB       │ Standard         │ $16                    │
│ RDS MySQL            │ db.t3.small      │ $26                    │
│ S3 Storage           │ 50 GB            │ $1                     │
│ Data Transfer        │ 10 GB out        │ $0.90                  │
│ Route 53             │ 1 hosted zone    │ $0.50                  │
│ CloudWatch           │ Basic monitoring │ $0                     │
├──────────────────────┼──────────────────┼────────────────────────┤
│ TOTAL ESTIMATE       │                  │ ~$104/month            │
└─────────────────────────────────────────────────────────────────┘

Azure Estimate:
┌─────────────────────────────────────────────────────────────────┐
│ Service              │ Specs            │ Monthly Cost           │
├──────────────────────┼──────────────────┼────────────────────────┤
│ VMs (2 instances)    │ B2s              │ $72  ($36 each)        │
│ Application Gateway  │ Standard v2 WAF  │ $117                   │
│ Azure SQL Database   │ S1               │ $15                    │
│ Blob Storage         │ 50 GB LRS        │ $1                     │
│ Data Transfer        │ 10 GB out        │ $0.87                  │
│ Azure DNS            │ 1 zone           │ $0.90                  │
│ Azure Monitor        │ Basic            │ $0                     │
├──────────────────────┼──────────────────┼────────────────────────┤
│ TOTAL ESTIMATE       │                  │ ~$207/month            │
│                      │ (App Gateway     │                        │
│                      │  expensive tier) │                        │
└─────────────────────────────────────────────────────────────────┘

⚠️ Use official calculators for accurate pricing:
   AWS:   https://calculator.aws
   Azure: https://azure.microsoft.com/pricing/calculator
```

---

## Summary: Key Concepts Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                   CLOUD COMPUTING CHEAT SHEET                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVICE MODELS                                                 │
│  IaaS = YOU manage OS, middleware, apps, data                   │
│  PaaS = YOU manage apps and data only                           │
│  SaaS = YOU manage nothing (just use it)                        │
│                                                                 │
│  DEPLOYMENT MODELS                                              │
│  Public  = Shared infrastructure (AWS, Azure, GCP)             │
│  Private = Dedicated infrastructure (your org only)             │
│  Hybrid  = Both public and private connected                    │
│  Multi   = Multiple cloud providers                             │
│                                                                 │
│  AWS KEY CONCEPTS                                               │
│  Region = Geographic area (us-east-1)                          │
│  AZ = Availability Zone = Data center within region            │
│  VPC = Virtual Private Cloud = Your private network            │
│  EC2 = Virtual machines                                         │
│  S3 = Object storage                                           │
│  IAM = Identity and Access Management                          │
│                                                                 │
│  AZURE KEY CONCEPTS                                             │
│  Region = Geographic area (East US)                            │
│  Availability Zone = Data centers within region                 │
│  VNet = Virtual Network = Your private network                 │
│  Virtual Machines = Compute instances                          │
│  Blob Storage = Object storage                                  │
│  Resource Group = Container for related resources              │
│  Azure AD/Entra = Identity management                          │
│  RBAC = Role-Based Access Control                              │
│                                                                 │
│  STORAGE TYPES                                                  │
│  Block  = Raw storage for databases (EBS/Azure Disks)          │
│  File   = Shared file system (EFS/Azure Files)                 │
│  Object = Unlimited file storage (S3/Blob Storage)             │
│                                                                 │
│  SECURITY BEST PRACTICES                                        │
│  ✅ Enable MFA on all accounts                                  │
│  ✅ Use IAM roles instead of access keys                        │
│  ✅ Follow principle of least privilege                         │
│  ✅ Keep sensitive data in private subnets                      │
│  ✅ Encrypt data at rest and in transit                         │
│  ✅ Enable logging and monitoring                               │
│  ✅ Use security groups to restrict access                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Learning Resources & Next Steps

```
┌─────────────────────────────────────────────────────────────────┐
│                    Recommended Learning Path                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BEGINNER (Months 1-2):                                        │
│  □ Complete this guide and set up accounts                     │
│  □ Launch EC2 instance and S3 bucket                           │
│  □ Create Azure VM and Storage Account                         │
│  □ Understand VPC/VNet networking basics                        │
│  □ Learn IAM/RBAC fundamentals                                 │
│                                                                 │
│  INTERMEDIATE (Months 3-4):                                    │
│  □ Deploy 3-tier application on AWS or Azure                   │
│  □ Set up auto-scaling and load balancing                      │
│  □ Implement monitoring and alerting                           │
│  □ Learn Infrastructure as Code (CloudFormation/Terraform)     │
│  □ Study AWS Solutions Architect Associate or                  │
│    Azure Administrator Associate certification                  │
│                                                                 │
│  ADVANCED (Months 5-6):                                        │
│  □ Container orchestration (Kubernetes/EKS/AKS)               │
│  □ Serverless architectures (Lambda/Azure Functions)           │
│  □ CI/CD pipelines (CodePipeline/Azure DevOps)                │
│  □ Multi-region disaster recovery                              │
│  □ Cost optimization strategies                                │
│                                                                 │
│  FREE PRACTICE RESOURCES:                                       │
│  AWS Free Tier: https://aws.amazon.com/free                    │
│  Azure Free:    https://azure.microsoft.com/free               │
│  AWS Training:  https://aws.amazon.com/training/digital        │
│  Microsoft Learn: https://learn.microsoft.com                  │
│  A Cloud Guru:  https://acloudguru.com (paid)                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*This guide covers foundational cloud computing concepts with manual implementation steps for both AWS and Azure. Practice each section in the free tiers before moving to production workloads. Always delete resources you're no longer using to avoid unexpected charges.*