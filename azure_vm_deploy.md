# Azure Virtual Machine Deployment - Manual Implementation Guide

## Complete Step-by-Step Process via Azure Portal

---

## Prerequisites

- Active Azure Subscription
- Azure Portal Access: [portal.azure.com](https://portal.azure.com)
- Basic understanding of networking concepts

---

## Architecture Overview

```
Internet
    │
    ▼
Public IP Address
    │
    ▼
Network Security Group (NSG)
    │
    ▼
Virtual Network (VNet)
    │
    ▼
Subnet
    │
    ▼
Network Interface Card (NIC)
    │
    ▼
Virtual Machine (VM)
    │
    ▼
Basic Web Application (HTML Page)
```

---

## Phase 1: Create Resource Group

### Step 1: Navigate to Resource Groups

```
Azure Portal Homepage
    → Click "Resource groups" (left sidebar or search bar)
    → Click "+ Create" button
```

### Step 2: Configure Resource Group

```
Project Details:
┌─────────────────────────────────────────────┐
│ Subscription    : [Your Subscription]        │
│ Resource Group  : my-vm-resource-group       │
│ Region          : East US                    │
└─────────────────────────────────────────────┘
```

### Step 3: Review and Create

```
→ Click "Review + Create"
→ Click "Create"
→ Wait for deployment confirmation ✓
```

---

## Phase 2: Create Virtual Network (VNet)

### Step 1: Navigate to Virtual Networks

```
Azure Portal
    → Search "Virtual Networks" in search bar
    → Click "Virtual Networks"
    → Click "+ Create"
```

### Step 2: Configure Basics Tab

```
┌─────────────────────────────────────────────┐
│ BASICS TAB                                   │
├─────────────────────────────────────────────┤
│ Subscription    : [Your Subscription]        │
│ Resource Group  : my-vm-resource-group       │
│ Name            : my-virtual-network         │
│ Region          : East US                    │
└─────────────────────────────────────────────┘
```

### Step 3: Configure IP Addresses Tab

```
┌─────────────────────────────────────────────┐
│ IP ADDRESSES TAB                             │
├─────────────────────────────────────────────┤
│ IPv4 address space : 10.0.0.0/16            │
│                                              │
│ Subnet Configuration:                        │
│   → Click "+ Add subnet"                    │
│   Subnet Name    : my-subnet                 │
│   Subnet Range   : 10.0.1.0/24              │
│   → Click "Add"                             │
└─────────────────────────────────────────────┘
```

### Step 4: Security Tab (Skip/Default)

```
→ Leave DDoS Protection: Basic
→ Leave Firewall: Disabled
→ Click "Next: Tags"
```

### Step 5: Create VNet

```
→ Click "Review + Create"
→ Verify all settings
→ Click "Create"
→ Wait for deployment ✓
```

---

## Phase 3: Create Public IP Address

### Step 1: Navigate to Public IP Addresses

```
Azure Portal
    → Search "Public IP addresses" in search bar
    → Click "Public IP addresses"
    → Click "+ Create"
```

### Step 2: Configure Public IP

```
┌─────────────────────────────────────────────┐
│ PUBLIC IP CONFIGURATION                      │
├─────────────────────────────────────────────┤
│ Subscription    : [Your Subscription]        │
│ Resource Group  : my-vm-resource-group       │
│ Region          : East US                    │
│ Name            : my-public-ip               │
│ IP Version      : IPv4                       │
│ SKU             : Standard                   │
│ Tier            : Regional                   │
│ Assignment      : Static                     │
│ Routing Pref    : Microsoft Network          │
│ Idle Timeout    : 4 minutes                  │
│ DNS Name Label  : my-vm-app-demo (optional)  │
└─────────────────────────────────────────────┘
```

### Step 3: Create Public IP

```
→ Click "Review + Create"
→ Click "Create"
→ Wait for deployment ✓
→ Note the assigned Public IP Address
```

---

## Phase 4: Create Network Security Group (NSG)

### Step 1: Navigate to NSG

```
Azure Portal
    → Search "Network Security Groups"
    → Click "Network Security Groups"
    → Click "+ Create"
```

### Step 2: Configure NSG Basics

```
┌─────────────────────────────────────────────┐
│ NSG BASICS                                   │
├─────────────────────────────────────────────┤
│ Subscription    : [Your Subscription]        │
│ Resource Group  : my-vm-resource-group       │
│ Name            : my-nsg                     │
│ Region          : East US                    │
└─────────────────────────────────────────────┘

→ Click "Review + Create"
→ Click "Create"
```

### Step 3: Configure Inbound Security Rules

```
After NSG is created:
    → Go to "my-nsg" resource
    → Click "Inbound security rules" (left sidebar)
    → Click "+ Add"
```

#### Rule 1: Allow SSH (Port 22)

```
┌─────────────────────────────────────────────┐
│ INBOUND RULE - SSH                           │
├─────────────────────────────────────────────┤
│ Source          : Any                        │
│ Source Port     : *                          │
│ Destination     : Any                        │
│ Service         : SSH                        │
│ Destination Port: 22                         │
│ Protocol        : TCP                        │
│ Action          : Allow                      │
│ Priority        : 100                        │
│ Name            : Allow-SSH                  │
└─────────────────────────────────────────────┘
→ Click "Add"
```

#### Rule 2: Allow HTTP (Port 80)

```
┌─────────────────────────────────────────────┐
│ INBOUND RULE - HTTP                          │
├─────────────────────────────────────────────┤
│ Source          : Any                        │
│ Source Port     : *                          │
│ Destination     : Any                        │
│ Service         : HTTP                       │
│ Destination Port: 80                         │
│ Protocol        : TCP                        │
│ Action          : Allow                      │
│ Priority        : 110                        │
│ Name            : Allow-HTTP                 │
└─────────────────────────────────────────────┘
→ Click "Add"
```

#### Rule 3: Allow HTTPS (Port 443)

```
┌─────────────────────────────────────────────┐
│ INBOUND RULE - HTTPS                         │
├─────────────────────────────────────────────┤
│ Source          : Any                        │
│ Source Port     : *                          │
│ Destination     : Any                        │
│ Service         : HTTPS                      │
│ Destination Port: 443                        │
│ Protocol        : TCP                        │
│ Action          : Allow                      │
│ Priority        : 120                        │
│ Name            : Allow-HTTPS                │
└─────────────────────────────────────────────┘
→ Click "Add"
```

---

## Phase 5: Create Virtual Machine

### Step 1: Navigate to Virtual Machines

```
Azure Portal
    → Click "Virtual Machines" (left sidebar)
    → Click "+ Create"
    → Select "Azure Virtual Machine"
```

### Step 2: Configure BASICS Tab

```
┌─────────────────────────────────────────────┐
│ PROJECT DETAILS                              │
├─────────────────────────────────────────────┤
│ Subscription    : [Your Subscription]        │
│ Resource Group  : my-vm-resource-group       │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ INSTANCE DETAILS                             │
├─────────────────────────────────────────────┤
│ VM Name         : my-linux-vm                │
│ Region          : East US                    │
│ Availability    : No infrastructure redundancy│
│ Security Type   : Standard                   │
│ Image           : Ubuntu Server 22.04 LTS    │
│ VM Architecture : x64                        │
│ Size            : Standard_B1s (1vCPU, 1GB)  │
│                   → Click "See all sizes"    │
│                   → Search "B1s"             │
│                   → Select Standard_B1s      │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ ADMINISTRATOR ACCOUNT                        │
├─────────────────────────────────────────────┤
│ Auth Type       : Password                   │
│ Username        : azureuser                  │
│ Password        : MySecurePass@123           │
│ Confirm Password: MySecurePass@123           │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ INBOUND PORT RULES                           │
├─────────────────────────────────────────────┤
│ Public Inbound  : Allow selected ports       │
│ Select Ports    : HTTP(80), HTTPS(443), SSH  │
└─────────────────────────────────────────────┘
```

### Step 3: Configure DISKS Tab

```
┌─────────────────────────────────────────────┐
│ DISK OPTIONS                                 │
├─────────────────────────────────────────────┤
│ OS Disk Type    : Standard SSD               │
│ OS Disk Size    : 30 GiB (Default)           │
│ Encryption      : Default                    │
└─────────────────────────────────────────────┘
→ Click "Next: Networking"
```

### Step 4: Configure NETWORKING Tab

```
┌─────────────────────────────────────────────┐
│ NETWORK INTERFACE                            │
├─────────────────────────────────────────────┤
│ Virtual Network : my-virtual-network         │
│ Subnet          : my-subnet (10.0.1.0/24)   │
│ Public IP       : my-public-ip               │
│ NIC NSG         : Advanced                   │
│ Configure NSG   : my-nsg                     │
│ Delete NIC with : ✓ Checked                  │
│ Delete IP with  : ✓ Checked                  │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ LOAD BALANCING                               │
├─────────────────────────────────────────────┤
│ Load Balancing  : None                       │
└─────────────────────────────────────────────┘
```

### Step 5: Configure MANAGEMENT Tab

```
┌─────────────────────────────────────────────┐
│ MONITORING                                   │
├─────────────────────────────────────────────┤
│ Boot Diagnostics: Enable with managed account│
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ AUTO-SHUTDOWN                                │
├─────────────────────────────────────────────┤
│ Enable Auto-shutdown: Enabled (Cost Saving)  │
│ Shutdown Time   : 11:00 PM                   │
│ Time Zone       : Your Time Zone             │
└─────────────────────────────────────────────┘
```

### Step 6: Configure ADVANCED Tab

```
→ Leave all settings as Default
→ Click "Next: Tags"
```

### Step 7: Add Tags (Optional)

```
┌─────────────────────────────────────────────┐
│ TAGS                                         │
├─────────────────────────────────────────────┤
│ Name       : Environment  Value: Development │
│ Name       : Project      Value: VM-Demo     │
│ Name       : Owner        Value: YourName    │
└─────────────────────────────────────────────┘
```

### Step 8: Review + Create

```
→ Click "Review + Create"
→ Review all configuration summary
→ Click "Create"
→ Wait for deployment (3-5 minutes) ✓
```

---

## Phase 6: Verify VM Deployment

### Step 1: Check VM Status

```
Azure Portal
    → Go to "Virtual Machines"
    → Click "my-linux-vm"
    → Verify Status: Running ✓
    → Note the Public IP Address
```

### Step 2: Verify Network Configuration

```
In VM Overview Page:
┌─────────────────────────────────────────────┐
│ VM DETAILS TO VERIFY                         │
├─────────────────────────────────────────────┤
│ Status          : Running                    │
│ Public IP       : XX.XX.XX.XX (your IP)     │
│ Private IP      : 10.0.1.4 (auto-assigned)  │
│ Virtual Network : my-virtual-network         │
│ Subnet          : my-subnet                  │
└─────────────────────────────────────────────┘
```

---

## Phase 7: Connect to VM and Host Application

### Step 1: Connect via Azure Portal (Browser SSH)

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Connect" (top menu)
    → Select "SSH"
    → Click "Open Azure Cloud Shell" 
    
    OR

    → Select "Connect via Bastion" (if configured)
    
    OR use any SSH Client:
    → Copy the SSH command shown
    → Open any SSH client (PuTTY, etc.)
```

### Step 2: Using Azure Serial Console (Alternative)

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Serial Console" (left sidebar under Support)
    → Wait for console to load
    → Login with credentials
```

### Step 3: Install Web Server via Azure Run Command

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Run command" (left sidebar)
    → Click "RunShellScript"
```

#### Script to Run in Run Command:

```bash
#!/bin/bash

# Update package list
apt-get update -y

# Install Nginx Web Server
apt-get install nginx -y

# Start and enable Nginx
systemctl start nginx
systemctl enable nginx

# Create custom HTML page
cat > /var/www/html/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Azure VM - Web Application</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #0078d4 0%, #00bcf2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .container {
            background: white;
            border-radius: 20px;
            padding: 50px;
            text-align: center;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 700px;
            width: 90%;
        }
        .azure-logo {
            font-size: 80px;
            margin-bottom: 20px;
        }
        h1 {
            color: #0078d4;
            font-size: 2.5em;
            margin-bottom: 15px;
        }
        .status-badge {
            background: #00b050;
            color: white;
            padding: 8px 20px;
            border-radius: 20px;
            font-size: 0.9em;
            display: inline-block;
            margin-bottom: 30px;
        }
        .info-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin: 30px 0;
        }
        .info-card {
            background: #f3f9ff;
            border: 2px solid #0078d4;
            border-radius: 10px;
            padding: 20px;
        }
        .info-card h3 {
            color: #0078d4;
            margin-bottom: 8px;
            font-size: 0.85em;
            text-transform: uppercase;
        }
        .info-card p {
            color: #333;
            font-size: 1em;
            font-weight: bold;
        }
        .footer {
            margin-top: 30px;
            color: #888;
            font-size: 0.85em;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="azure-logo">☁️</div>
        <h1>Azure VM Successfully Deployed!</h1>
        <div class="status-badge">✓ Application Running</div>
        <p style="color:#555; margin-bottom:20px;">
            This web application is hosted on Microsoft Azure Virtual Machine
        </p>
        <div class="info-grid">
            <div class="info-card">
                <h3>Cloud Provider</h3>
                <p>Microsoft Azure</p>
            </div>
            <div class="info-card">
                <h3>Web Server</h3>
                <p>Nginx</p>
            </div>
            <div class="info-card">
                <h3>Operating System</h3>
                <p>Ubuntu 22.04 LTS</p>
            </div>
            <div class="info-card">
                <h3>Status</h3>
                <p style="color:#00b050">● Live & Running</p>
            </div>
        </div>
        <div class="footer">
            <p>Deployed via Azure Portal | Azure Virtual Machine Demo</p>
        </div>
    </div>
</body>
</html>
EOF

# Set correct permissions
chmod 644 /var/www/html/index.html

# Restart Nginx to apply changes
systemctl restart nginx

echo "Web application deployed successfully!"
```

```
→ Paste script in the script box
→ Click "Run"
→ Wait for execution (2-3 minutes)
→ Verify output: "Web application deployed successfully!"
```

---

## Phase 8: Verify Public Access

### Step 1: Get Public IP

```
Azure Portal
    → Go to "my-linux-vm"
    → Overview page
    → Copy "Public IP Address": XX.XX.XX.XX
```

### Step 2: Test Application

```
Open any Web Browser:
    → Type: http://XX.XX.XX.XX
    → Press Enter
    → You should see the Azure VM Web Application page ✓
```

### Step 3: Verify in Azure Portal

```
Azure Portal - Verify Connectivity:
    → VM → Networking → Check NSG rules active
    → VM → Overview → Status: Running
    → Public IP → Associated to NIC ✓
```

---

## Phase 9: Monitor Your VM

### Step 1: Check VM Metrics

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Metrics" (left sidebar)
    → Add Metric:
        • Percentage CPU
        • Network In Total
        • Network Out Total
        • Disk Read Bytes
```

### Step 2: Check Activity Log

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Activity log" (left sidebar)
    → View all operations performed
```

### Step 3: Check Boot Diagnostics

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Boot diagnostics" (left sidebar)
    → Click "Screenshot" to verify VM boot
```

---

## Resource Summary

```
┌─────────────────────────────────────────────────────────────┐
│              DEPLOYED RESOURCES SUMMARY                      │
├─────────────────┬───────────────────────────────────────────┤
│ Resource Type   │ Name                                       │
├─────────────────┼───────────────────────────────────────────┤
│ Resource Group  │ my-vm-resource-group                       │
│ Virtual Network │ my-virtual-network (10.0.0.0/16)          │
│ Subnet          │ my-subnet (10.0.1.0/24)                   │
│ Public IP       │ my-public-ip (Static, Standard)            │
│ NSG             │ my-nsg (SSH, HTTP, HTTPS rules)            │
│ Virtual Machine │ my-linux-vm (Ubuntu 22.04, Standard_B1s)  │
│ OS Disk         │ my-linux-vm_OsDisk (Standard SSD)          │
│ NIC             │ my-linux-vm-nic                            │
│ Web Server      │ Nginx                                      │
│ Application     │ Custom HTML Web Page                       │
└─────────────────┴───────────────────────────────────────────┘
```

---

## DHCP & IP Assignment Explanation

```
┌─────────────────────────────────────────────────────────────┐
│              HOW IP ASSIGNMENT WORKS IN AZURE               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PRIVATE IP (Internal):                                     │
│  • Azure DHCP automatically assigns private IP              │
│  • Range: 10.0.1.0/24 → VM gets 10.0.1.4                  │
│  • First 3 IPs (.1, .2, .3) reserved by Azure              │
│  • VM does NOT know its public IP                           │
│                                                             │
│  PUBLIC IP (External):                                      │
│  • Assigned as Static (won't change on restart)            │
│  • Mapped to NIC via NAT by Azure                          │
│  • Traffic flows: Internet → Public IP → NAT → Private IP  │
│                                                             │
│  Flow:                                                      │
│  User Browser → XX.XX.XX.XX:80 (Public IP)                 │
│      → Azure NAT → 10.0.1.4:80 (Private IP)               │
│      → VM Nginx → Serves HTML Page                         │
│      → Response back to User ✓                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Cost Management

### Stop VM When Not in Use

```
Azure Portal
    → Go to "my-linux-vm"
    → Click "Stop" (top menu)
    → Confirm stop
    → Status changes to: Stopped (Deallocated)
    → Compute charges STOP ✓
    → Storage charges continue (minimal)
```

### Delete All Resources (Cleanup)

```
Azure Portal
    → Go to "my-vm-resource-group"
    → Click "Delete resource group"
    → Type resource group name to confirm
    → Click "Delete"
    → ALL resources deleted ✓ (No more charges)
```

---

## Troubleshooting

```
┌─────────────────────────────────────────────────────────────┐
│ ISSUE              │ SOLUTION                               │
├────────────────────┼────────────────────────────────────────┤
│ Can't access HTTP  │ Check NSG - Port 80 rule exists        │
│                    │ Check VM is Running state              │
│                    │ Verify Public IP is associated         │
├────────────────────┼────────────────────────────────────────┤
│ SSH not working    │ Check NSG - Port 22 rule exists        │
│                    │ Verify correct username/password       │
├────────────────────┼────────────────────────────────────────┤
│ Website not loading│ Run command - check nginx status       │
│                    │ Verify nginx is running                │
├────────────────────┼────────────────────────────────────────┤
│ VM not starting    │ Check Boot Diagnostics                 │
│                    │ Check Activity Log for errors          │
└────────────────────┴────────────────────────────────────────┘
```

---

## ✅ Deployment Complete!

```
SUCCESS CHECKLIST:
□ Resource Group Created
□ Virtual Network Created with Subnet
□ Public IP Address Created (Static)
□ NSG Created with SSH, HTTP, HTTPS Rules
□ Virtual Machine Deployed (Ubuntu 22.04)
□ Nginx Web Server Installed
□ Custom HTML Application Hosted
□ Application Accessible via Public IP
□ http://XX.XX.XX.XX → Shows Web Page ✓
```