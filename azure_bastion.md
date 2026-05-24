# Azure Virtual Networking: Theory & Implementation Guide

## Table of Contents
1. [Introduction to Azure Virtual Networking](#introduction)
2. [Core Concepts & Theory](#core-concepts)
3. [Network Architecture Design](#network-architecture)
4. [Azure Bastion Host](#azure-bastion)
5. [Implementation Steps](#implementation-steps)
6. [Architecture Diagram](#architecture-diagram)
7. [Best Practices](#best-practices)
8. [Summary](#summary)

---

## 1. Introduction to Azure Virtual Networking

Azure Virtual Networking is the **fundamental building block** for private networks in Microsoft Azure. It enables Azure resources like Virtual Machines (VMs), databases, and web apps to securely communicate with each other, the internet, and on-premises networks.

### Why Virtual Networking?
- Isolate and protect cloud resources
- Control inbound and outbound traffic
- Enable hybrid cloud connectivity
- Implement zero-trust security model

---

## 2. Core Concepts & Theory

### 2.1 Virtual Network (VNet)
A **VNet** is a logically isolated network in Azure that closely resembles a traditional on-premises network.

| Property | Description |
|----------|-------------|
| Address Space | CIDR block assigned to the VNet (e.g., `10.0.0.0/16`) |
| Region | VNet is bound to a specific Azure region |
| Subscription | VNet belongs to a single Azure subscription |
| Isolation | Complete isolation from other VNets by default |

### 2.2 Subnets
Subnets **divide a VNet** into smaller, manageable segments.

```
VNet: 10.0.0.0/16
├── Subnet-Web:     10.0.1.0/24   (Public-facing)
├── Subnet-App:     10.0.2.0/24   (Application layer)
├── Subnet-DB:      10.0.3.0/24   (Database layer)
└── AzureBastionSubnet: 10.0.4.0/27 (Bastion - Reserved Name)
```

**Key Subnet Facts:**
- Azure reserves **5 IP addresses** per subnet (first 4 + last 1)
- Subnets in same VNet can communicate by default
- Each subnet can have its own Network Security Group (NSG)

### 2.3 Network Security Groups (NSG)
NSGs act as a **virtual firewall** controlling traffic to/from Azure resources.

#### NSG Rule Components:
| Component | Description | Example |
|-----------|-------------|---------|
| Priority | Lower number = higher priority | 100, 200, 300 |
| Source | Origin of traffic | IP, CIDR, Service Tag |
| Destination | Target of traffic | IP, CIDR, Any |
| Protocol | Traffic protocol | TCP, UDP, ICMP, Any |
| Port Range | Port or range | 80, 443, 3389, 22 |
| Action | Allow or Deny | Allow / Deny |

#### Default NSG Rules (Inbound):
```
Priority    Name                        Action
65000       AllowVNetInBound            Allow
65001       AllowAzureLoadBalancerInBound Allow
65500       DenyAllInBound              Deny
```

### 2.4 Public vs Private IP Addresses

| Type | Scope | Use Case | Persistence |
|------|-------|----------|-------------|
| Public IP (Dynamic) | Internet-facing | Dev/Test VMs | Changes on stop/start |
| Public IP (Static) | Internet-facing | Production, DNS | Never changes |
| Private IP (Dynamic) | Internal VNet | Backend servers | May change |
| Private IP (Static) | Internal VNet | Domain controllers, DBs | Fixed |

### 2.5 Azure DNS
- Each VNet gets an **Azure-provided DNS** automatically
- Default DNS: `168.63.129.16`
- Custom DNS servers can be configured
- VM hostnames resolvable within VNet

### 2.6 VNet Peering
Connects **two VNets** allowing resources to communicate as if in same network.

```
VNet-A (10.0.0.0/16) <----Peering----> VNet-B (10.1.0.0/16)
```

**Types:**
- **Regional Peering** — Same Azure region
- **Global Peering** — Different Azure regions

### 2.7 Network Routing
Azure automatically creates **system routes** for traffic flow.

```
Route Table Example:
Destination     Next Hop Type
0.0.0.0/0       Internet
10.0.0.0/16     VirtualNetwork
10.0.1.0/24     VirtualNetwork
```

---

## 3. Network Architecture Design

### 3.1 Hub-Spoke Architecture (Recommended)

```
                    ┌─────────────────────────┐
                    │      HUB VNet           │
                    │    (10.0.0.0/16)        │
                    │  ┌─────────────────┐    │
                    │  │  Azure Bastion  │    │
                    │  │  Azure Firewall │    │
                    │  │  VPN Gateway    │    │
                    │  └─────────────────┘    │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    ┌─────────▼──────┐  ┌────────▼───────┐  ┌──────▼──────────┐
    │  Spoke VNet 1  │  │  Spoke VNet 2  │  │  Spoke VNet 3   │
    │  (Production)  │  │  (Development) │  │   (Testing)     │
    │  10.1.0.0/16   │  │  10.2.0.0/16  │  │  10.3.0.0/16    │
    └────────────────┘  └───────────────┘  └─────────────────┘
```

### 3.2 Three-Tier Architecture (Common Pattern)

```
Internet
    │
    ▼
┌─────────────────────────────────────┐
│           VNet (10.0.0.0/16)        │
│                                     │
│  ┌──────────────────────────────┐   │
│  │  Web Tier Subnet             │   │
│  │  10.0.1.0/24                 │   │
│  │  [VM-Web1] [VM-Web2]         │   │
│  │  NSG: Allow 80, 443 Inbound  │   │
│  └──────────────┬───────────────┘   │
│                 │                   │
│  ┌──────────────▼───────────────┐   │
│  │  App Tier Subnet             │   │
│  │  10.0.2.0/24                 │   │
│  │  [VM-App1] [VM-App2]         │   │
│  │  NSG: Allow 8080 from Web    │   │
│  └──────────────┬───────────────┘   │
│                 │                   │
│  ┌──────────────▼───────────────┐   │
│  │  DB Tier Subnet              │   │
│  │  10.0.3.0/24                 │   │
│  │  [SQL Server] [MySQL]        │   │
│  │  NSG: Allow 1433 from App    │   │
│  └──────────────────────────────┘   │
│                                     │
│  ┌──────────────────────────────┐   │
│  │  AzureBastionSubnet          │   │
│  │  10.0.4.0/27                 │   │
│  │  [Azure Bastion Host]        │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

### 3.3 IP Address Planning

```
Organization VNet Plan:
├── VNet Address Space: 10.0.0.0/8 (Large Enterprise)
│
├── Production:  10.1.0.0/16
│   ├── Web:     10.1.1.0/24  → 251 usable hosts
│   ├── App:     10.1.2.0/24  → 251 usable hosts
│   └── DB:      10.1.3.0/24  → 251 usable hosts
│
├── Development: 10.2.0.0/16
│   ├── Web:     10.2.1.0/24
│   └── App:     10.2.2.0/24
│
└── Bastion:     10.1.4.0/27  → 27 usable hosts (min /27 for Bastion)
```

---

## 4. Azure Bastion Host

### 4.1 What is Azure Bastion?
Azure Bastion is a **fully managed PaaS service** that provides secure and seamless RDP/SSH connectivity to VMs **directly from the Azure portal** over TLS (port 443) — without exposing VMs to the public internet.

### 4.2 Traditional vs Bastion Access

```
❌ TRADITIONAL (Insecure):
User ──(Public Internet)──► VM Public IP ──► RDP/SSH Port Open
                                              (Attack Surface!)

✅ WITH BASTION (Secure):
User ──(HTTPS/443)──► Azure Bastion ──► VM Private IP ──► RDP/SSH
                      (Managed PaaS)    (No Public IP on VM!)
```

### 4.3 How Azure Bastion Works

```
┌────────────────────────────────────────────────────────────┐
│                     Azure Portal / Browser                  │
│                    (HTML5 Web Client)                       │
└───────────────────────────┬────────────────────────────────┘
                            │ HTTPS (Port 443)
                            ▼
┌────────────────────────────────────────────────────────────┐
│              AzureBastionSubnet (10.0.4.0/27)              │
│                                                            │
│          ┌─────────────────────────────────┐               │
│          │        Azure Bastion Host       │               │
│          │   - Public IP (for user access) │               │
│          │   - TLS Termination             │               │
│          │   - Session Management          │               │
│          └──────────────┬──────────────────┘               │
│                         │                                  │
└─────────────────────────┼──────────────────────────────────┘
                          │ RDP (3389) / SSH (22) - Internal
                          ▼
┌────────────────────────────────────────────────────────────┐
│              VM Subnet (10.0.1.0/24)                       │
│          ┌─────────────────────────────────┐               │
│          │    Virtual Machine              │               │
│          │    Private IP: 10.0.1.4         │               │
│          │    No Public IP Required! ✅    │               │
│          └─────────────────────────────────┘               │
└────────────────────────────────────────────────────────────┘
```

### 4.4 Azure Bastion SKUs

| Feature | Basic SKU | Standard SKU |
|---------|-----------|--------------|
| RDP/SSH via Portal | ✅ | ✅ |
| Custom Ports | ❌ | ✅ |
| Native Client Support | ❌ | ✅ |
| IP-based Connection | ❌ | ✅ |
| Shareable Links | ❌ | ✅ |
| File Transfer | ❌ | ✅ |
| Scale Units | 2 | 2–50 |
| Price | Lower | Higher |

### 4.5 Bastion Security Benefits
- ✅ No RDP/SSH ports exposed to internet
- ✅ No need for Public IPs on VMs
- ✅ Protection against port scanning
- ✅ Zero-day RDP/SSH vulnerability protection
- ✅ Session recording & audit logs (Standard)
- ✅ MFA support through Azure AD
- ✅ SSL/TLS encrypted sessions

### 4.6 Required NSG Rules for Bastion

**Bastion Subnet NSG — Inbound Rules:**
```
Priority  Source              Port   Destination  Action
100       Internet            443    Bastion       Allow
110       GatewayManager      443    Bastion       Allow  ← Required!
120       AzureLoadBalancer   443    Bastion       Allow  ← Required!
```

**Bastion Subnet NSG — Outbound Rules:**
```
Priority  Source   Port        Destination    Action
100       Bastion  3389, 22    VirtualNetwork Allow  ← RDP/SSH to VMs
110       Bastion  443         AzureCloud     Allow  ← Azure services
```

---

## 5. Implementation Steps

### Prerequisites
- Active Azure Subscription
- Azure Portal access or Azure CLI installed
- Contributor or Owner role on the subscription

---

### Step 1: Create a Resource Group

#### Via Azure Portal:
```
1. Login → portal.azure.com
2. Search → "Resource Groups"
3. Click → "+ Create"
4. Fill Details:
   - Subscription: Your Subscription
   - Resource Group Name: RG-NetworkDemo
   - Region: East US (or your preferred region)
5. Click → "Review + Create" → "Create"
```

#### Via Azure CLI:
```bash
# Login to Azure
az login

# Create Resource Group
az group create \
  --name RG-NetworkDemo \
  --location eastus

# Verify creation
az group show --name RG-NetworkDemo
```

---

### Step 2: Create Virtual Network with Subnets

#### Via Azure Portal:
```
1. Search → "Virtual Networks"
2. Click → "+ Create"
3. Basics Tab:
   - Resource Group: RG-NetworkDemo
   - Name: VNet-Main
   - Region: East US
4. IP Addresses Tab:
   - IPv4 Address Space: 10.0.0.0/16
   - Delete default subnet
   - Add Subnet → Web-Subnet
     └── Address Range: 10.0.1.0/24
   - Add Subnet → App-Subnet
     └── Address Range: 10.0.2.0/24
   - Add Subnet → DB-Subnet
     └── Address Range: 10.0.3.0/24
   - Add Subnet → AzureBastionSubnet  ← Exact name required!
     └── Address Range: 10.0.4.0/27
5. Click → "Review + Create" → "Create"
```

#### Via Azure CLI:
```bash
# Create Virtual Network
az network vnet create \
  --resource-group RG-NetworkDemo \
  --name VNet-Main \
  --address-prefix 10.0.0.0/16 \
  --location eastus

# Create Web Subnet
az network vnet subnet create \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name Web-Subnet \
  --address-prefix 10.0.1.0/24

# Create App Subnet
az network vnet subnet create \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name App-Subnet \
  --address-prefix 10.0.2.0/24

# Create DB Subnet
az network vnet subnet create \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name DB-Subnet \
  --address-prefix 10.0.3.0/24

# Create Bastion Subnet (Name must be exactly AzureBastionSubnet)
az network vnet subnet create \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name AzureBastionSubnet \
  --address-prefix 10.0.4.0/27

# Verify VNet and Subnets
az network vnet show \
  --resource-group RG-NetworkDemo \
  --name VNet-Main \
  --output table
```

---

### Step 3: Create Network Security Groups (NSG)

#### Create NSG for Web Tier:
```bash
# Create Web NSG
az network nsg create \
  --resource-group RG-NetworkDemo \
  --name NSG-Web \
  --location eastus

# Allow HTTP (Port 80)
az network nsg rule create \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-Web \
  --name Allow-HTTP \
  --priority 100 \
  --protocol Tcp \
  --destination-port-range 80 \
  --access Allow \
  --direction Inbound

# Allow HTTPS (Port 443)
az network nsg rule create \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-Web \
  --name Allow-HTTPS \
  --priority 110 \
  --protocol Tcp \
  --destination-port-range 443 \
  --access Allow \
  --direction Inbound

# Deny all other Inbound
az network nsg rule create \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-Web \
  --name Deny-All-Inbound \
  --priority 4096 \
  --protocol '*' \
  --destination-port-range '*' \
  --access Deny \
  --direction Inbound
```

#### Create NSG for DB Tier:
```bash
# Create DB NSG
az network nsg create \
  --resource-group RG-NetworkDemo \
  --name NSG-DB \
  --location eastus

# Allow SQL from App Subnet only
az network nsg rule create \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-DB \
  --name Allow-SQL-From-App \
  --priority 100 \
  --protocol Tcp \
  --source-address-prefix 10.0.2.0/24 \
  --destination-port-range 1433 \
  --access Allow \
  --direction Inbound

# Deny all other Inbound
az network nsg rule create \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-DB \
  --name Deny-All \
  --priority 4096 \
  --protocol '*' \
  --destination-port-range '*' \
  --access Deny \
  --direction Inbound
```

#### Associate NSGs to Subnets:
```bash
# Associate Web NSG to Web Subnet
az network vnet subnet update \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name Web-Subnet \
  --network-security-group NSG-Web

# Associate DB NSG to DB Subnet
az network vnet subnet update \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --name DB-Subnet \
  --network-security-group NSG-DB
```

---

### Step 4: Create Virtual Machine (Without Public IP)

#### Via Azure Portal:
```
1. Search → "Virtual Machines"
2. Click → "+ Create" → "Azure Virtual Machine"
3. Basics Tab:
   - Resource Group: RG-NetworkDemo
   - VM Name: VM-Web01
   - Region: East US
   - Image: Windows Server 2022 / Ubuntu 22.04
   - Size: Standard_B2s
   - Username: azureadmin
   - Password: [Strong Password]
   
4. Networking Tab:
   - Virtual Network: VNet-Main
   - Subnet: Web-Subnet (10.0.1.0/24)
   - Public IP: NONE  ← Important for Bastion!
   - NIC NSG: None (we created custom NSG)
   
5. Click → "Review + Create" → "Create"
```

#### Via Azure CLI:
```bash
# Create VM without Public IP
az vm create \
  --resource-group RG-NetworkDemo \
  --name VM-Web01 \
  --image Ubuntu2204 \
  --vnet-name VNet-Main \
  --subnet Web-Subnet \
  --public-ip-address "" \
  --admin-username azureadmin \
  --admin-password "SecurePassword123!" \
  --size Standard_B2s \
  --location eastus

# Verify VM - should show no public IP
az vm show \
  --resource-group RG-NetworkDemo \
  --name VM-Web01 \
  --show-details \
  --query "{Name:name, PrivateIP:privateIps, PublicIP:publicIps}" \
  --output table
```

---

### Step 5: Deploy Azure Bastion Host

#### Via Azure Portal:
```
1. Search → "Bastions"
2. Click → "+ Create"
3. Basics Tab:
   - Resource Group: RG-NetworkDemo
   - Name: Bastion-Main
   - Region: East US (Same as VNet!)
   - Tier: Basic (or Standard)
   - Virtual Network: VNet-Main
   - Subnet: AzureBastionSubnet (auto-detected)
   - Public IP Address: Create New
     └── Name: Bastion-PublicIP
4. Click → "Review + Create" → "Create"
   ⚠️ Deployment takes 5-10 minutes
```

#### Via Azure CLI:
```bash
# Create Public IP for Bastion
az network public-ip create \
  --resource-group RG-NetworkDemo \
  --name Bastion-PublicIP \
  --sku Standard \
  --location eastus

# Create Azure Bastion
az network bastion create \
  --resource-group RG-NetworkDemo \
  --name Bastion-Main \
  --public-ip-address Bastion-PublicIP \
  --vnet-name VNet-Main \
  --location eastus \
  --sku Basic

# Check Bastion deployment status
az network bastion show \
  --resource-group RG-NetworkDemo \
  --name Bastion-Main \
  --query "provisioningState"
```

---

### Step 6: Connect to VM via Azure Bastion

#### Via Azure Portal:
```
1. Navigate to → Virtual Machines → VM-Web01
2. Click → "Connect" in the top menu
3. Select → "Bastion" tab
4. Click → "Use Bastion"
5. Enter Credentials:
   - Username: azureadmin
   - Password: [Your Password]
6. Click → "Connect"
   ✅ Browser opens RDP/SSH session securely!
```

#### Via Azure CLI (Standard SKU only):
```bash
# SSH via Bastion using native client
az network bastion ssh \
  --name Bastion-Main \
  --resource-group RG-NetworkDemo \
  --target-resource-id $(az vm show -g RG-NetworkDemo -n VM-Web01 --query id -o tsv) \
  --auth-type password \
  --username azureadmin

# RDP via Bastion (Windows VM)
az network bastion rdp \
  --name Bastion-Main \
  --resource-group RG-NetworkDemo \
  --target-resource-id $(az vm show -g RG-NetworkDemo -n VM-Web01 --query id -o tsv)
```

---

### Step 7: Verify Network Configuration

```bash
# List all resources in Resource Group
az resource list \
  --resource-group RG-NetworkDemo \
  --output table

# Check VNet details
az network vnet show \
  --resource-group RG-NetworkDemo \
  --name VNet-Main

# List all subnets
az network vnet subnet list \
  --resource-group RG-NetworkDemo \
  --vnet-name VNet-Main \
  --output table

# Check NSG rules
az network nsg rule list \
  --resource-group RG-NetworkDemo \
  --nsg-name NSG-Web \
  --output table

# Check effective security rules on VM NIC
az network nic show-effective-nsg \
  --resource-group RG-NetworkDemo \
  --name VM-Web01VMNic \
  --output table
```

---

### Step 8: Cleanup Resources (Optional)

```bash
# Delete entire Resource Group (removes all resources)
az group delete \
  --name RG-NetworkDemo \
  --yes \
  --no-wait

# Verify deletion
az group exists --name RG-NetworkDemo
```

---

## 6. Architecture Diagram

```
                          INTERNET
                              │
                              │ HTTPS (443) Only
                              ▼
╔═════════════════════════════════════════════════════════════╗
║              AZURE REGION: East US                          ║
║                                                             ║
║  ┌─────────────────────────────────────────────────────┐   ║
║  │           VNet-Main (10.0.0.0/16)                   │   ║
║  │                                                     │   ║
║  │  ┌──────────────────────────────────────────────┐   │   ║
║  │  │  AzureBastionSubnet (10.0.4.0/27)            │   │   ║
║  │  │  ┌────────────────────────────────────┐      │   │   ║
║  │  │  │     🔒 Azure Bastion Host          │      │   │   ║
║  │  │  │     Public IP: 20.x.x.x           │      │   │   ║
║  │  │  │     TLS Termination               │      │   │   ║
║  │  │  └────────────┬───────────────────────┘      │   │   ║
║  │  └───────────────┼──────────────────────────────┘   │   ║
║  │                  │ RDP/SSH (Internal)                │   ║
║  │  ┌───────────────▼──────────────────────────────┐   │   ║
║  │  │  Web-Subnet (10.0.1.0/24) [NSG-Web]          │   │   ║
║  │  │  ┌─────────────┐  ┌─────────────┐            │   │   ║
║  │  │  │  VM-Web01   │  │  VM-Web02   │            │   │   ║
║  │  │  │ 10.0.1.4    │  │ 10.0.1.5    │            │   │   ║
║  │  │  │ No Pub IP ✅│  │ No Pub IP ✅│            │   │   ║
║  │  │  └─────────────┘  └─────────────┘            │   │   ║
║  │  └───────────────────────┬──────────────────────┘   │   ║
║  │                          │ Port 8080                 │   ║
║  │  ┌───────────────────────▼──────────────────────┐   │   ║
║  │  │  App-Subnet (10.0.2.0/24) [NSG-App]          │   │   ║
║  │  │  ┌─────────────┐  ┌─────────────┐            │   │   ║
║  │  │  │  VM-App01   │  │  VM-App02   │            │   │   ║
║  │  │  │ 10.0.2.4    │  │ 10.0.2.5    │            │   │   ║
║  │  │  └─────────────┘  └─────────────┘            │   │   ║
║  │  └───────────────────────┬──────────────────────┘   │   ║
║  │                          │ Port 1433                 │   ║
║  │  ┌───────────────────────▼──────────────────────┐   │   ║
║  │  │  DB-Subnet (10.0.3.0/24) [NSG-DB]            │   │   ║
║  │  │  ┌─────────────────────────────────┐          │   │   ║
║  │  │  │       SQL Server / MySQL        │          │   │   ║
║  │  │  │       10.0.3.4                  │          │   │   ║
║  │  │  └─────────────────────────────────┘          │   │   ║
║  │  └──────────────────────────────────────────────┘   │   ║
║  └─────────────────────────────────────────────────────┘   ║
╚═════════════════════════════════════════════════════════════╝
```

---

## 7. Best Practices

### 7.1 VNet Design
```
✅ DO:
  - Plan IP address space before deployment
  - Use non-overlapping CIDR blocks
  - Leave room for future growth
  - Use separate subnets for each tier
  - Enable DDoS Protection for production

❌ DON'T:
  - Use public IP ranges (1.0.0.0/8) for VNets
  - Create VNets too small (/29 or smaller)
  - Mix workloads in same subnet without NSG
  - Expose RDP/SSH ports to internet
```

### 7.2 NSG Best Practices
```
✅ DO:
  - Apply NSGs at subnet level (not just NIC)
  - Use Service Tags instead of IP ranges
  - Document all NSG rules
  - Use Deny-All as last rule
  - Regular audit of NSG rules

✅ Service Tags to Use:
  - Internet        → All internet traffic
  - VirtualNetwork  → All VNet traffic
  - AzureCloud      → Azure datacenter IPs
  - GatewayManager  → Azure Gateway manager
```

### 7.3 Azure Bastion Best Practices
```
✅ DO:
  - Use Standard SKU for production
  - Enable diagnostic logs
  - Use /27 or larger for BastionSubnet
  - Enable MFA for Azure portal access
  - Regularly review Bastion access logs

❌ DON'T:
  - Deploy other resources in AzureBastionSubnet
  - Remove required NSG rules for Bastion
  - Use Basic SKU for enterprise deployments
```

### 7.4 Security Checklist

| Item | Status |
|------|--------|
| VMs have no public IP | ✅ |
| NSGs applied to all subnets | ✅ |
| RDP/SSH not open to internet | ✅ |
| Azure Bastion deployed | ✅ |
| DDoS Protection enabled | ✅ |
| Network Watcher enabled | ✅ |
| Diagnostic logs configured | ✅ |
| VNet Flow Logs enabled | ✅ |

---

## 8. Summary

### What We Implemented:

```
COMPONENT           RESOURCE NAME          VALUE
─────────────────────────────────────────────────────
Resource Group    → RG-NetworkDemo
Virtual Network   → VNet-Main             10.0.0.0/16
Web Subnet        → Web-Subnet            10.0.1.0/24
App Subnet        → App-Subnet            10.0.2.0/24
DB Subnet         → DB-Subnet             10.0.3.0/24
Bastion Subnet    → AzureBastionSubnet    10.0.4.0/27
Web NSG           → NSG-Web              Allow 80, 443
DB NSG            → NSG-DB               Allow 1433 from App
Virtual Machine   → VM-Web01             No Public IP
Bastion Host      → Bastion-Main         Secure RDP/SSH
```

### Key Takeaways:
1. **VNet** provides network isolation in Azure
2. **Subnets** segment the network for security tiers
3. **NSGs** act as virtual firewalls with priority-based rules
4. **Azure Bastion** eliminates need for public IPs on VMs
5. **Zero Public Exposure** — VMs accessible only via Bastion
6. **Defense in Depth** — Multiple security layers implemented

---

## References

| Resource | Link |
|----------|------|
| Azure VNet Documentation | [docs.microsoft.com/azure/virtual-network](https://docs.microsoft.com/azure/virtual-network) |
| Azure Bastion Documentation | [docs.microsoft.com/azure/bastion](https://docs.microsoft.com/azure/bastion) |
| NSG Documentation | [docs.microsoft.com/azure/network-security-groups](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview) |
| Azure CLI Reference | [docs.microsoft.com/cli/azure](https://docs.microsoft.com/cli/azure) |
| Azure Pricing Calculator | [azure.microsoft.com/pricing/calculator](https://azure.microsoft.com/pricing/calculator) |

---

*Documentation prepared for Azure Virtual Networking Lab*  
*Last Updated: 2024*  
*Author: Azure Network Implementation Guide*
```