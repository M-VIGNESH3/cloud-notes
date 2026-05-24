# Azure Hybrid Cloud - Hub & Spoke with On-Premises VPN Connectivity

## Complete End-to-End Documentation

---

```markdown
# 🌐 Azure Hybrid Cloud Architecture
## Hub-and-Spoke with On-Premises VPN Connectivity
### Complete Implementation Guide

**Author:** DevOps & Cloud Engineering Team  
**Version:** 1.0  
**Last Updated:** 2025  
**Status:** Implementation Guide

---

## 📋 Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Prerequisites](#2-prerequisites)
3. [Azure Infrastructure Setup](#3-azure-infrastructure-setup)
4. [On-Premises Router VM Setup (VMware)](#4-on-premises-router-vm-setup-vmware)
5. [Network Configuration](#5-network-configuration)
6. [VPN Configuration](#6-vpn-configuration)
7. [Azure VPN Connection](#7-azure-vpn-connection)
8. [StrongSwan IPsec Setup](#8-strongswan-ipsec-setup)
9. [Testing & Verification](#9-testing--verification)
10. [Troubleshooting Guide](#10-troubleshooting-guide)
11. [Security Best Practices](#11-security-best-practices)
12. [Architecture Summary](#12-architecture-summary)

---

## 1. Architecture Overview

### 1.1 What We Are Building

This documentation covers the complete implementation of a
**Hybrid Cloud Architecture** that connects:

- ✅ Azure Hub-and-Spoke Network
- ✅ On-Premises Simulated Network (VMware Ubuntu)
- ✅ Secure Site-to-Site VPN Tunnel
- ✅ Centralized Security via Azure Firewall
- ✅ Zero-Trust Access via Azure Bastion

---

### 1.2 High-Level Architecture Diagram

```
╔══════════════════════════════════════════════════════════════════╗
║                        AZURE CLOUD                               ║
║                                                                  ║
║   ┌─────────────────────────────────────────────────────────┐   ║
║   │                    HUB VNET (vnet1)                     │   ║
║   │                                                         │   ║
║   │   ┌─────────────────┐    ┌─────────────────┐           │   ║
║   │   │  Azure Firewall │    │  Azure Bastion  │           │   ║
║   │   │  azure-firewall │    │  azure-bastion  │           │   ║
║   │   │  [Public IP]    │    │  [Public IP]    │           │   ║
║   │   └────────┬────────┘    └────────┬────────┘           │   ║
║   │            │                      │                     │   ║
║   │   ┌────────┴──────────────────────┴──────┐             │   ║
║   │   │         VPN Gateway (vng)            │             │   ║
║   │   │         [Public IP: vng-ip]          │             │   ║
║   │   └──────────────────┬───────────────────┘             │   ║
║   └──────────────────────│─────────────────────────────────┘   ║
║                          │                                       ║
║   ┌──────────────────────│─────────────────────────────────┐   ║
║   │   SPOKE1 (vnet2)     │    SPOKE2 (vnet3)               │   ║
║   │                      │                                  │   ║
║   │   ┌──────────────┐   │    ┌──────────────┐             │   ║
║   │   │  vm-vn3l2    │   │    │  vm-vnet3    │             │   ║
║   │   │  [Cloud VM]  │   │    │  [Bridge VM] │             │   ║
║   │   │  NSG Applied │   │    │  NSG Applied │             │   ║
║   │   └──────────────┘   │    └──────────────┘             │   ║
║   └──────────────────────│─────────────────────────────────┘   ║
║                          │                                       ║
╚══════════════════════════│═══════════════════════════════════════╝
                           │
                    [IPsec VPN Tunnel]
                    [Encrypted Traffic]
                           │
╔══════════════════════════│═══════════════════════════════════════╗
║              ON-PREMISES (VMware Simulated)                      ║
║                          │                                       ║
║   ┌──────────────────────│─────────────────────────────────┐   ║
║   │              Ubuntu Router VM                           │   ║
║   │                                                         │   ║
║   │   ens33 (WAN) ───────┘    ens37 (LAN)                 │   ║
║   │   [DHCP - Internet]        [172.16.0.1/24]             │   ║
║   │                                                         │   ║
║   │   NAT + IP Forwarding + StrongSwan IPsec               │   ║
║   └─────────────────────────────────────────────────────────┘   ║
║                                                                  ║
║   ┌─────────────────────────────────────────────────────────┐   ║
║   │           On-Premises Internal Network                  │   ║
║   │              172.16.0.0/24                              │   ║
║   │                                                         │   ║
║   │   [On-Prem Clients/Servers]                             │   ║
║   └─────────────────────────────────────────────────────────┘   ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

### 1.3 Traffic Flow Diagrams

#### Cloud-to-Cloud Communication
```
vm-vn3l2 ──► vnet2 ──► route2 ──► azure-firewall ──► vnet1/vnet3
```

#### Cloud-to-On-Premises Communication
```
vm-vn3l2 ──► vnet2 ──► azure-firewall ──► vng ──► VPN Tunnel ──► Ubuntu Router ──► 172.16.0.0/24
```

#### Administrative Access
```
Administrator ──► Internet ──► azure-bastion ──► vm-vn3l2 / vm-vnet3
```

#### Outbound Internet Traffic
```
vm-vn3l2 ──► azure-firewall ──► azure-firewall-ip ──► Internet
```

#### On-Premises to Azure Communication
```
On-Prem Client ──► Ubuntu Router (ens37) ──► StrongSwan ──► ens33 ──► VPN Tunnel ──► vng ──► azure-firewall ──► Azure VMs
```

---

### 1.4 Component Summary Table

| Component | Type | Purpose | Location |
|-----------|------|---------|----------|
| `vnet1` | Azure VNet | Hub Network | Azure |
| `vnet2` | Azure VNet | Spoke - Cloud Workloads | Azure |
| `vnet3` | Azure VNet | Spoke - Bridge to On-Prem | Azure |
| `azure-firewall` | Azure Firewall | Central Traffic Inspection | Hub (vnet1) |
| `azure-bastion` | Azure Bastion | Secure Remote Access | Hub (vnet1) |
| `vng` | VPN Gateway | Site-to-Site VPN | Hub (vnet1) |
| `vm-vn3l2` | Azure VM | Cloud Workload VM | vnet2 |
| `vm-vnet3` | Azure VM | Bridge/DMZ VM | vnet3 |
| `vm-vn3l2-nsg` | NSG | Cloud VM Security | vnet2 |
| `vm-vnet3-nsg` | NSG | Bridge VM Security | vnet3 |
| `route2` | Route Table | Traffic Routing | vnet2 |
| `route3` | Route Table | On-Prem Traffic Routing | vnet3 |
| Ubuntu Router | VMware VM | On-Prem Router/VPN | On-Premises |

---

## 2. Prerequisites

### 2.1 Azure Requirements

```
✅ Azure Subscription (with contributor access)
✅ Resource Group created
✅ Azure Region selected (consistent across all resources)
✅ Sufficient vCPU quota for VMs
✅ VPN Gateway SKU support (VpnGw1 minimum)
```

### 2.2 On-Premises Requirements

```
✅ VMware Workstation / VMware Player installed
✅ Ubuntu Server 22.04 LTS ISO downloaded
✅ Minimum VM specs:
   - CPU  : 2 vCPUs
   - RAM  : 2 GB
   - Disk : 20 GB
   - NICs : 2 (NAT + Host-Only or NAT + NAT)
✅ Internet connectivity on host machine
```

### 2.3 Network Planning

```
Azure VNets:
┌─────────────────────────────────────────┐
│ vnet1 (Hub)    : 10.1.0.0/16           │
│ vnet2 (Spoke1) : 10.2.0.0/16           │
│ vnet3 (Spoke2) : 10.3.0.0/16           │
│                                         │
│ GatewaySubnet  : 10.1.0.0/27           │
│ AzureFirewall  : 10.1.1.0/26           │
│ AzureBastion   : 10.1.2.0/27           │
└─────────────────────────────────────────┘

On-Premises:
┌─────────────────────────────────────────┐
│ LAN Network    : 172.16.0.0/24         │
│ Router LAN IP  : 172.16.0.1            │
│ Router WAN IP  : DHCP (from VMware)    │
└─────────────────────────────────────────┘
```

---

## 3. Azure Infrastructure Setup

### 3.1 Resource Group

```
Azure Portal → Resource Groups → Create

Name     : rg-hybrid-network
Region   : East US (or your preferred region)
Tags     : Environment=Production, Project=HybridCloud
```

---

### 3.2 Virtual Networks

#### Hub VNet (vnet1)

```
Azure Portal → Virtual Networks → Create

Basic:
  Name            : vnet1
  Region          : East US
  Resource Group  : rg-hybrid-network

IP Addresses:
  Address Space   : 10.1.0.0/16

Subnets:
  ┌──────────────────────────────────────────┐
  │ Subnet Name         │ Address Range      │
  ├──────────────────────────────────────────┤
  │ GatewaySubnet       │ 10.1.0.0/27        │
  │ AzureFirewallSubnet │ 10.1.1.0/26        │
  │ AzureBastionSubnet  │ 10.1.2.0/27        │
  │ hub-general-subnet  │ 10.1.3.0/24        │
  └──────────────────────────────────────────┘
```

#### Spoke VNet (vnet2)

```
Azure Portal → Virtual Networks → Create

Basic:
  Name            : vnet2
  Region          : East US
  Resource Group  : rg-hybrid-network

IP Addresses:
  Address Space   : 10.2.0.0/16

Subnets:
  ┌──────────────────────────────────────────┐
  │ Subnet Name         │ Address Range      │
  ├──────────────────────────────────────────┤
  │ workload-subnet     │ 10.2.1.0/24        │
  └──────────────────────────────────────────┘
```

#### Spoke VNet (vnet3)

```
Azure Portal → Virtual Networks → Create

Basic:
  Name            : vnet3
  Region          : East US
  Resource Group  : rg-hybrid-network

IP Addresses:
  Address Space   : 10.3.0.0/16

Subnets:
  ┌──────────────────────────────────────────┐
  │ Subnet Name         │ Address Range      │
  ├──────────────────────────────────────────┤
  │ onprem-bridge-subnet│ 10.3.1.0/24        │
  └──────────────────────────────────────────┘
```

---

### 3.3 VNet Peering (Hub-Spoke)

```
Peering 1: vnet1 ←→ vnet2
─────────────────────────────────────────────
Azure Portal → vnet1 → Peerings → Add

  Peering Name (vnet1→vnet2) : hub-to-spoke1
  Remote VNet                : vnet2
  Peering Name (vnet2→vnet1) : spoke1-to-hub
  
  Settings:
  ✅ Allow gateway transit (on vnet1 side)
  ✅ Use remote gateway    (on vnet2 side)
  ✅ Allow forwarded traffic

Peering 2: vnet1 ←→ vnet3
─────────────────────────────────────────────
  Peering Name (vnet1→vnet3) : hub-to-spoke2
  Remote VNet                : vnet3
  Peering Name (vnet3→vnet1) : spoke2-to-hub
  
  Settings:
  ✅ Allow gateway transit (on vnet1 side)
  ✅ Use remote gateway    (on vnet3 side)
  ✅ Allow forwarded traffic
```

---

### 3.4 Azure Firewall

```
Azure Portal → Firewalls → Create

Basic:
  Name            : azure-firewall
  Region          : East US
  Resource Group  : rg-hybrid-network
  SKU             : Standard
  VNet            : vnet1
  Subnet          : AzureFirewallSubnet (10.1.1.0/26)
  Public IP       : Create New → azure-firewall-ip
```

#### Firewall Policy Rules

```
Azure Portal → Firewall Policies → Create → azure-fw-policy

Network Rules:
┌─────────────────────────────────────────────────────────────────┐
│ Rule Collection: allow-spoke-to-spoke                           │
│ Priority: 100                                                   │
├──────────────┬──────────┬───────────┬──────────┬───────────────┤
│ Name         │ Protocol │ Source    │ Dest     │ Dest Port     │
├──────────────┼──────────┼───────────┼──────────┼───────────────┤
│ allow-vnet2  │ Any      │ 10.2.0.0  │ 10.3.0.0 │ *             │
│ to-vnet3     │          │ /16       │ /16      │               │
├──────────────┼──────────┼───────────┼──────────┼───────────────┤
│ allow-vnet2  │ Any      │ 10.2.0.0  │ 172.16.0 │ *             │
│ to-onprem    │          │ /16       │ .0/24    │               │
├──────────────┼──────────┼───────────┼──────────┼───────────────┤
│ allow-onprem │ Any      │ 172.16.0  │ 10.0.0.0 │ *             │
│ to-azure     │          │ .0/24     │ /8       │               │
└──────────────┴──────────┴───────────┴──────────┴───────────────┘

Application Rules:
┌─────────────────────────────────────────────────────────────────┐
│ Rule Collection: allow-internet                                 │
│ Priority: 200                                                   │
├──────────────┬──────────┬───────────┬──────────────────────────┤
│ Name         │ Protocol │ Source    │ Target FQDN              │
├──────────────┼──────────┼───────────┼──────────────────────────┤
│ allow-web    │ HTTP/S   │ 10.0.0.0  │ *                        │
│              │          │ /8        │                          │
└──────────────┴──────────┴───────────┴──────────────────────────┘
```

---

### 3.5 Azure Bastion

```
Azure Portal → Bastions → Create

Basic:
  Name            : azure-bastion
  Region          : East US
  Resource Group  : rg-hybrid-network
  VNet            : vnet1
  Subnet          : AzureBastionSubnet (10.1.2.0/27)
  Public IP       : Create New → azure-bastion-ip
  SKU             : Basic
```

---

### 3.6 VPN Gateway

```
Azure Portal → Virtual Network Gateways → Create

Basic:
  Name            : vng
  Region          : East US
  Gateway Type    : VPN
  VPN Type        : Route-based
  SKU             : VpnGw1
  Generation      : Generation1
  VNet            : vnet1
  Subnet          : GatewaySubnet (10.1.0.0/27)
  Public IP       : Create New → vng-ip
  Active-Active   : Disabled
  BGP             : Disabled

⚠️  NOTE: VPN Gateway takes 30-45 minutes to deploy
```

---

### 3.7 Route Tables

#### Route Table for vnet2 (route2)

```
Azure Portal → Route Tables → Create

Name            : route2
Region          : East US
Resource Group  : rg-hybrid-network

Routes:
┌───────────────────────────────────────────────────────────────┐
│ Route Name      │ Address Prefix  │ Next Hop Type  │ Next Hop │
├───────────────────────────────────────────────────────────────┤
│ to-hub-firewall │ 0.0.0.0/0       │ VirtualAppl.   │ 10.1.1.4 │
│ to-onprem       │ 172.16.0.0/24   │ VirtualAppl.   │ 10.1.1.4 │
│ to-vnet3        │ 10.3.0.0/16     │ VirtualAppl.   │ 10.1.1.4 │
└───────────────────────────────────────────────────────────────┘

Associate to: vnet2 → workload-subnet
```

#### Route Table for vnet3 (route3)

```
Azure Portal → Route Tables → Create

Name            : route3
Region          : East US
Resource Group  : rg-hybrid-network

Routes:
┌───────────────────────────────────────────────────────────────┐
│ Route Name      │ Address Prefix  │ Next Hop Type  │ Next Hop │
├───────────────────────────────────────────────────────────────┤
│ to-hub-firewall │ 0.0.0.0/0       │ VirtualAppl.   │ 10.1.1.4 │
│ to-onprem       │ 172.16.0.0/24   │ VirtualAppl.   │ 10.1.1.4 │
│ to-vnet2        │ 10.2.0.0/16     │ VirtualAppl.   │ 10.1.1.4 │
└───────────────────────────────────────────────────────────────┘

Associate to: vnet3 → onprem-bridge-subnet
```

---

### 3.8 Network Security Groups

#### NSG for Cloud VM (vm-vn3l2-nsg)

```
Azure Portal → Network Security Groups → Create

Name: vm-vn3l2-nsg

Inbound Rules:
┌──────────────────────────────────────────────────────────────────┐
│ Priority │ Name           │ Port  │ Protocol │ Source  │ Action  │
├──────────────────────────────────────────────────────────────────┤
│ 100      │ allow-bastion  │ 3389  │ TCP      │ Bastion │ Allow   │
│          │                │ 22    │ TCP      │ Subnet  │         │
├──────────────────────────────────────────────────────────────────┤
│ 200      │ allow-internal │ *     │ Any      │ 10.0.0  │ Allow   │
│          │                │       │          │ .0/8    │         │
├──────────────────────────────────────────────────────────────────┤
│ 4096     │ deny-all       │ *     │ Any      │ *       │ Deny    │
└──────────────────────────────────────────────────────────────────┘

Associate to: vm-vn3l2 network interface
```

#### NSG for Bridge VM (vm-vnet3-nsg)

```
Name: vm-vnet3-nsg

Inbound Rules:
┌──────────────────────────────────────────────────────────────────┐
│ Priority │ Name           │ Port  │ Protocol │ Source  │ Action  │
├──────────────────────────────────────────────────────────────────┤
│ 100      │ allow-bastion  │ 22    │ TCP      │ Bastion │ Allow   │
│          │                │       │          │ Subnet  │         │
├──────────────────────────────────────────────────────────────────┤
│ 200      │ allow-vpn      │ *     │ Any      │ 172.16  │ Allow   │
│          │                │       │          │ .0.0/24 │         │
├──────────────────────────────────────────────────────────────────┤
│ 300      │ allow-internal │ *     │ Any      │ 10.0.0  │ Allow   │
│          │                │       │          │ .0/8    │         │
├──────────────────────────────────────────────────────────────────┤
│ 4096     │ deny-all       │ *     │ Any      │ *       │ Deny    │
└──────────────────────────────────────────────────────────────────┘

Associate to: vm-vnet3 network interface
```

---

### 3.9 Azure Virtual Machines

#### Cloud VM (vm-vn3l2)

```
Azure Portal → Virtual Machines → Create

Basic:
  Name            : vm-vn3l2
  Region          : East US
  Image           : Ubuntu Server 22.04 LTS
  Size            : Standard_B2s
  Auth Type       : SSH Public Key
  Username        : azureuser

Networking:
  VNet            : vnet2
  Subnet          : workload-subnet
  Public IP       : None (access via Bastion)
  NSG             : vm-vn3l2-nsg

Disks:
  OS Disk         : vm-vn3l2_disk1_a75e8e95...
  Type            : Standard SSD
```

#### Bridge VM (vm-vnet3)

```
Azure Portal → Virtual Machines → Create

Basic:
  Name            : vm-vnet3
  Region          : East US
  Image           : Ubuntu Server 22.04 LTS
  Size            : Standard_B2s
  Auth Type       : SSH Public Key
  Username        : azureuser

Networking:
  VNet            : vnet3
  Subnet          : onprem-bridge-subnet
  Public IP       : None (access via Bastion)
  NSG             : vm-vnet3-nsg

Disks:
  OS Disk         : vm-vnet3_disk1_3bea63b1...
  Type            : Standard SSD
```

---

## 4. On-Premises Router VM Setup (VMware)

### 4.1 VMware VM Configuration

```
VMware Workstation → Create New Virtual Machine

VM Settings:
┌──────────────────────────────────────────┐
│ OS        : Ubuntu Server 22.04 LTS      │
│ CPU       : 2 vCPUs                      │
│ RAM       : 2048 MB                      │
│ Disk      : 20 GB                        │
│                                          │
│ Network Adapters:                        │
│  NIC 1 (ens33) : NAT                    │
│  NIC 2 (ens37) : NAT / Host-Only        │
└──────────────────────────────────────────┘

⚠️  IMPORTANT: Two NICs are required
    NIC1 = WAN (Internet/Azure connectivity)
    NIC2 = LAN (On-premises internal network)
```

### 4.2 VMware Network Adapter Layout

```
┌─────────────────────────────────────────────────────────┐
│                  VMware Host Machine                    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Ubuntu Router VM                   │   │
│  │                                                  │   │
│  │  ┌──────────────┐      ┌──────────────┐         │   │
│  │  │ ens33 (WAN)  │      │ ens37 (LAN)  │         │   │
│  │  │ DHCP         │      │ 172.16.0.1   │         │   │
│  │  └──────┬───────┘      └──────┬───────┘         │   │
│  └─────────│────────────────────│─────────────────┘   │
│            │                    │                       │
│   ┌────────▼────────┐  ┌────────▼────────┐             │
│   │  VMnet8 (NAT)   │  │  VMnet1/Custom  │             │
│   │  Internet Access│  │  Internal LAN   │             │
│   └────────┬────────┘  └─────────────────┘             │
└────────────│────────────────────────────────────────────┘
             │
         [Internet]
             │
         [Azure VPN]
```

---

## 5. Network Configuration

### 5.1 Ubuntu Installation

```
1. Boot Ubuntu Server 22.04 LTS ISO in VMware
2. Follow installation wizard
3. When prompted for network:
   - Configure ens33 with DHCP
   - Skip ens37 (configure manually later)
4. Complete installation and reboot
```

### 5.2 Verify Interfaces After Boot

```bash
# Step 1: Check available interfaces
ip a

# Expected Output:
# ─────────────────────────────────────────────────
# 1: lo: <LOOPBACK,UP,LOWER_UP>
#    inet 127.0.0.1/8
#
# 2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP>
#    inet 192.168.x.x/24  ← DHCP assigned (WAN)
#
# 3: ens37: <BROADCAST,MULTICAST>
#    ← No IP yet (LAN - we will configure this)
# ─────────────────────────────────────────────────
```

### 5.3 Configure Netplan

```bash
# Step 2: View existing netplan files
ls /etc/netplan

# Step 3: Edit configuration
sudo nano /etc/netplan/00-installer-config.yaml
```

**Complete Netplan Configuration:**

```yaml
# /etc/netplan/00-installer-config.yaml
# ─────────────────────────────────────────────────────
# On-Premises Router VM - Network Configuration
# 
# ens33 → WAN Interface (NAT → Internet → Azure VPN)
# ens37 → LAN Interface (On-premises internal network)
# ─────────────────────────────────────────────────────

network:
  version: 2
  renderer: networkd

  ethernets:

    # ── WAN Interface ──────────────────────────────
    ens33:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 100       # Preferred default route
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

    # ── LAN Interface ──────────────────────────────
    ens37:
      dhcp4: false
      addresses:
        - 172.16.0.1/24         # Router LAN IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

```bash
# Step 4: Validate configuration (no output = no errors)
sudo netplan generate

# Step 5: Apply configuration
sudo netplan apply

# Step 6: Verify routing table (should show 3 routes)
ip route show

# Expected Output:
# ─────────────────────────────────────────────────────
# default via 192.168.x.x dev ens33 proto dhcp metric 100
# 172.16.0.0/24 dev ens37 proto kernel scope link src 172.16.0.1
# 192.168.x.0/24 dev ens33 proto kernel scope link src 192.168.x.x
# ─────────────────────────────────────────────────────
```

---

### 5.4 Enable IP Forwarding

```bash
# Step 7: Create sysctl configuration
sudo nano /etc/sysctl.d/99-onprem-router.conf
```

```bash
# /etc/sysctl.d/99-onprem-router.conf
# ─────────────────────────────────────────────────
# IP Forwarding & Routing Configuration
# ─────────────────────────────────────────────────

# Enable IPv4 packet forwarding (required for routing)
net.ipv4.ip_forward = 1

# Disable ICMP redirects (security best practice for routers)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Disable reverse path filtering (required for VPN)
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
```

```bash
# Step 8: Apply sysctl settings
sudo sysctl --system

# Step 9: Verify IP forwarding is enabled
sysctl net.ipv4.ip_forward
# Expected: net.ipv4.ip_forward = 1
```

---

### 5.5 Configure NAT with iptables

```bash
# Step 10: Add NAT masquerade (internet access for LAN clients)
sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE

# Step 11: Allow forwarding between interfaces
sudo iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
sudo iptables -A FORWARD -i ens33 -o ens37 \
  -m state --state RELATED,ESTABLISHED -j ACCEPT

# Step 12: Verify NAT rules
sudo iptables -t nat -L -n -v

# Step 13: Save rules permanently
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

---

### 5.6 Verify Internet Connectivity

```bash
# Step 14: Test internet connectivity
ping -c 4 8.8.8.8

# Step 15: Test DNS resolution
ping -c 4 google.com

# Step 16: Get public IP (needed for Azure configuration)
curl -4 ifconfig.me
# Note this IP - you will need it for Azure Local Network Gateway
```

---

## 6. VPN Configuration

### 6.1 VPN Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                     VPN TUNNEL DETAILS                           │
├──────────────────────────────────────────────────────────────────┤
│ Protocol          : IPsec/IKEv2                                  │
│ Authentication    : Pre-Shared Key (PSK)                         │
│ Encryption (IKE)  : AES-256 / SHA-256 / DH Group 2 (modp1024)  │
│ Encryption (ESP)  : AES-256 / SHA-256                           │
│ Local Network     : 172.16.0.0/24 (On-premises)                 │
│ Remote Network    : 10.0.0.0/8   (Azure VNets)                  │
│ DPD               : Enabled (30s delay, 120s timeout)            │
│ Auto-reconnect    : Enabled                                      │
└──────────────────────────────────────────────────────────────────┘
```

### 6.2 IKEv2 Handshake Flow

```
Ubuntu (On-Prem)                    Azure VPN Gateway
      │                                     │
      │──── IKE_SA_INIT (Propose) ─────────►│
      │◄─── IKE_SA_INIT (Accept) ───────────│
      │                                     │
      │──── IKE_AUTH (Identity + PSK) ─────►│
      │◄─── IKE_AUTH (Success) ─────────────│
      │                                     │
      │══════ IPsec Tunnel Established ═════│
      │                                     │
      │──── CREATE_CHILD_SA (Traffic) ─────►│
      │◄─── CREATE_CHILD_SA (Confirm) ──────│
      │                                     │
      │═══════════ Data Flows ══════════════│
```

---

## 7. Azure VPN Connection

### 7.1 Get On-Premises Public IP

```bash
# Run on Ubuntu Router VM
curl -4 ifconfig.me
# Example output: 203.0.113.50
# ⚠️  Note this IP - required for next step
```

### 7.2 Create Local Network Gateway

```
Azure Portal → Local Network Gateways → Create

Basic:
  Name            : onprem-lgw
  Region          : East US
  Resource Group  : rg-hybrid-network

Endpoint:
  IP Address      : <YOUR_PUBLIC_IP from curl ifconfig.me>

Address Space:
  172.16.0.0/24   ← Your on-premises LAN

Tags:
  Environment     : Production
  Purpose         : OnPremises-Gateway
```

### 7.3 Create VPN Connection

```
Azure Portal → Virtual Network Gateways → vng
→ Connections → Add

Settings:
  Name                : azure-to-onprem
  Connection Type     : Site-to-Site (IPsec)
  VPN Gateway         : vng
  Local Network GW    : onprem-lgw
  Shared Key (PSK)    : <YourStrongPSKHere2024!>
  IKE Protocol        : IKEv2
  
⚠️  Save the PSK - you will need it for StrongSwan

Custom IPsec Policy (Optional but Recommended):
  IKE Phase 1:
    Encryption      : AES256
    Integrity       : SHA256
    DH Group        : DHGroup2
    
  IKE Phase 2:
    Encryption      : AES256
    Integrity       : SHA256
    PFS Group       : None
    SA Lifetime     : 3600 seconds
```

### 7.4 Note Azure VPN Gateway Public IP

```
Azure Portal → Virtual Network Gateways → vng
→ Overview → Public IP Address

Note: XX.XX.XX.XX  ← You need this for StrongSwan config
```

---

## 8. StrongSwan IPsec Setup

### 8.1 Install StrongSwan

```bash
# Step 1: Update packages
sudo apt update && sudo apt upgrade -y

# Step 2: Install StrongSwan and plugins
sudo apt install -y \
  strongswan \
  strongswan-pki \
  libcharon-extra-plugins \
  libcharon-extauth-plugins \
  libstrongswan-extra-plugins

# Step 3: Verify installation
ipsec version
# Expected: Linux StrongSwan 5.x.x
```

---

### 8.2 Configure IPsec (ipsec.conf)

```bash
# Backup original
sudo cp /etc/ipsec.conf /etc/ipsec.conf.backup

# Edit configuration
sudo nano /etc/ipsec.conf
```

```bash
# /etc/ipsec.conf
# ─────────────────────────────────────────────────────────────────
# StrongSwan IPsec Configuration
# On-Premises Ubuntu Router → Azure VPN Gateway
# ─────────────────────────────────────────────────────────────────

config setup
    charondebug="ike 2, knl 2, cfg 2, net 2, esp 2, dmn 2, mgr 2"
    uniqueids=no

# ── Default Connection Parameters ────────────────────────────────
conn %default
    ikelifetime=28800s              # IKE SA lifetime  : 8 hours
    keylife=3600s                   # IPsec SA lifetime : 1 hour
    rekeymargin=3m                  # Rekey before expiry
    keyingtries=%forever            # Keep retrying forever
    authby=secret                   # Pre-shared key auth
    keyexchange=ikev2               # IKEv2 (matches Azure)
    mobike=no                       # Disable MOBIKE

# ── Azure Site-to-Site VPN Connection ────────────────────────────
conn azure-s2s-vpn

    # Local (On-Premises) Side
    left=%defaultroute              # Auto-detect local IP
    leftid=<YOUR_PUBLIC_IP>         # ← Replace: curl ifconfig.me
    leftsubnet=172.16.0.0/24        # On-premises LAN subnet
    leftfirewall=yes                # Enable firewall traversal

    # Remote (Azure) Side
    right=<AZURE_VNG_PUBLIC_IP>     # ← Replace: Azure vng-ip
    rightid=<AZURE_VNG_PUBLIC_IP>   # ← Same as right
    rightsubnet=10.0.0.0/8          # Azure VNet address space
    # Adjust if your Azure VNets are different:
    # rightsubnet=10.1.0.0/16,10.2.0.0/16,10.3.0.0/16

    # Connection Behavior
    auto=start                      # Start on boot
    closeaction=restart             # Restart if connection drops
    dpdaction=restart               # Restart on dead peer detection
    dpddelay=30s                    # DPD check every 30 seconds
    dpdtimeout=120s                 # DPD timeout after 120 seconds

    # Encryption Settings (Must match Azure custom policy)
    ike=aes256-sha256-modp1024!     # Phase 1 : IKE encryption
    esp=aes256-sha256!              # Phase 2 : ESP encryption
```

---

### 8.3 Configure Pre-Shared Key (ipsec.secrets)

```bash
# Edit secrets file
sudo nano /etc/ipsec.secrets
```

```bash
# /etc/ipsec.secrets
# ─────────────────────────────────────────────────────────────────
# IPsec Pre-Shared Key Configuration
# Format: LOCAL_ID REMOTE_ID : PSK "shared-key"
# ─────────────────────────────────────────────────────────────────

<YOUR_PUBLIC_IP> <AZURE_VNG_PUBLIC_IP> : PSK "<YourStrongPSKHere2024!>"

# ─────────────────────────────────────────────────────────────────
# ⚠️  IMPORTANT:
#   - PSK must EXACTLY match what you set in Azure connection
#   - Replace both IP placeholders with actual values
#   - Keep this file secure (chmod 600)
# ─────────────────────────────────────────────────────────────────
```

```bash
# Secure the file
sudo chmod 600 /etc/ipsec.secrets

# Verify permissions
ls -la /etc/ipsec.secrets
# Expected: -rw------- 1 root root
```

---

### 8.4 Configure iptables for VPN Traffic

```bash
# Allow IKE (VPN negotiation)
sudo iptables -A INPUT -p udp --dport 500 -j ACCEPT

# Allow NAT-Traversal (VPN behind NAT)
sudo iptables -A INPUT -p udp --dport 4500 -j ACCEPT

# Allow ESP (encrypted VPN data)
sudo iptables -A INPUT -p esp -j ACCEPT

# Allow forwarding for VPN tunnel traffic
sudo iptables -A FORWARD -i ens33 -o ens33 \
  -m policy --dir in --pol ipsec -j ACCEPT

sudo iptables -A FORWARD -o ens33 \
  -m policy --dir out --pol ipsec -j ACCEPT

# Save all rules
sudo netfilter-persistent save
```

---

### 8.5 Start StrongSwan

```bash
# Start StrongSwan service
sudo systemctl start strongswan-starter

# Enable on boot
sudo systemctl enable strongswan-starter

# Check service status
sudo systemctl status strongswan-starter

# Bring up VPN connection
sudo ipsec up azure-s2s-vpn

# Check connection status
sudo ipsec status
sudo ipsec statusall
```

---

## 9. Testing & Verification

### 9.1 Verification Checklist

```
PHASE 1 - Network Basics
────────────────────────────────────────────────────────
□ ip a          → ens33 has DHCP IP, ens37 has 172.16.0.1
□ ip route      → 3 routes visible (default, LAN, WAN subnet)
□ ping 8.8.8.8  → Internet reachable
□ ping google   → DNS resolution working
□ ifconfig.me   → Public IP returned

PHASE 2 - VPN Tunnel
────────────────────────────────────────────────────────
□ ipsec status         → Tunnel shows ESTABLISHED
□ Azure Portal         → Connection shows Connected ✅
□ ping Azure VM IP     → Reachable from Ubuntu
□ ping 172.16.0.1      → Reachable from Azure VM (via Bastion)

PHASE 3 - End-to-End
────────────────────────────────────────────────────────
□ Azure VM → On-prem  → Traffic flows through firewall
□ On-prem → Azure VM  → Traffic flows through VPN + firewall
□ Bastion access       → Can reach both VMs via Bastion
```

---

### 9.2 Verification Commands

```bash
# ── On Ubuntu Router VM ──────────────────────────────────────────

# Check interfaces
ip a
ip link show

# Check routes (expect 3 entries)
ip route show

# Check VPN tunnel status
sudo ipsec status
sudo ipsec statusall

# Test internet
ping -c 4 8.8.8.8

# Test Azure VM connectivity
ping -c 4 <AZURE-VM-PRIVATE-IP>

# Test DNS
nslookup google.com

# Monitor VPN logs live
sudo journalctl -u strongswan-starter -f

# Check iptables rules
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v
```

```bash
# ── On Azure VM (via Bastion) ─────────────────────────────────────

# Test connectivity to on-premises router
ping -c 4 172.16.0.1

# Trace route to on-premises
traceroute 172.16.0.1

# Check if traffic goes through firewall
traceroute 172.16.0.1
# Should pass through: Azure VM → Firewall → VPN GW → Ubuntu Router
```

---

### 9.3 Expected Test Results

```
Test: ping 8.8.8.8 (from Ubuntu)
──────────────────────────────────
PING 8.8.8.8 56 bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=12.3ms ✅

Test: sudo ipsec status
──────────────────────────────────
Security Associations (1 up, 0 connecting):
azure-s2s-vpn[1]: ESTABLISHED 00:05:23 ago ✅
azure-s2s-vpn{1}: INSTALLED, TUNNEL ✅
azure-s2s-vpn{1}: 172.16.0.0/24 === 10.0.0.0/8

Test: Azure Portal Connection Status
──────────────────────────────────
azure-to-onprem : Connected ✅
Data In         : XX KB
Data Out        : XX KB
```

---

## 10. Troubleshooting Guide

### 10.1 Issue: Only 1 Route Showing in ip route

```
Symptom: ip route shows only 1 entry instead of 3
─────────────────────────────────────────────────────────────
Cause 1: ens37 not configured properly in netplan
Cause 2: renderer set to NetworkManager instead of networkd
Cause 3: netplan not applied after changes

Fix:
─────────────────────────────────────────────────────────────
# Check netplan config
sudo cat /etc/netplan/00-installer-config.yaml

# Ensure renderer is networkd (not NetworkManager)
# Ensure ens37 has addresses configured

# Re-apply
sudo netplan generate
sudo netplan apply

# Verify
ip route show
```

### 10.2 Issue: Cannot ping 8.8.8.8

```
Symptom: ping 8.8.8.8 fails / times out
─────────────────────────────────────────────────────────────
Cause 1: No default gateway configured
Cause 2: VMware NAT not working
Cause 3: Firewall blocking ICMP

Fix:
─────────────────────────────────────────────────────────────
# Check default route exists
ip route | grep default
# Should show: default via 192.168.x.x dev ens33

# If missing, add temporarily
sudo ip route add default via <VMWARE_NAT_GATEWAY>

# Check VMware NAT service
# On VMware Host: Edit → Virtual Network Editor
# Ensure VMnet8 (NAT) is configured

# Check iptables not blocking
sudo iptables -L INPUT -n -v
sudo iptables -L FORWARD -n -v
```

### 10.3 Issue: VPN Tunnel Not Establishing

```
Symptom: ipsec status shows CONNECTING but not ESTABLISHED
─────────────────────────────────────────────────────────────
Cause 1: PSK mismatch between Azure and StrongSwan
Cause 2: Wrong Azure VPN Gateway IP in ipsec.conf
Cause 3: Firewall blocking UDP 500/4500
Cause 4: IKEv2 encryption mismatch

Fix:
─────────────────────────────────────────────────────────────
# Check logs for errors
sudo journalctl -u strongswan-starter --since "5 minutes ago"
sudo tail -f /var/log/syslog | grep charon

# Verify PSK matches exactly
sudo cat /etc/ipsec.secrets

# Verify Azure VPN Gateway IP
ping <AZURE_VNG_IP>

# Check ports are open
sudo netstat -ulnp | grep -E '500|4500'

# Restart and retry
sudo ipsec restart
sudo ipsec up azure-s2s-vpn
```

### 10.4 Issue: Can Ping Azure VPN GW but Not Azure VMs

```
Symptom: VPN is up but cannot reach Azure VM private IPs
─────────────────────────────────────────────────────────────
Cause 1: rightsubnet in ipsec.conf doesn't match Azure VNet
Cause 2: Azure Firewall blocking traffic
Cause 3: Azure NSG blocking traffic
Cause 4: Route table missing on Azure side

Fix:
─────────────────────────────────────────────────────────────
# Verify rightsubnet covers your Azure VM IPs
sudo cat /etc/ipsec.conf | grep rightsubnet

# Check Azure Firewall rules in portal
# → Allow on-premises (172.16.0.0/24) to Azure VMs

# Check NSG rules on destination Azure VM
# → Allow inbound from 172.16.0.0/24

# Check route tables in Azure
# → Ensure on-prem route points to VPN Gateway
```

### 10.5 Common Error Messages

```
Error: "received AUTHENTICATION_FAILED notify error"
Fix  : PSK mismatch → check /etc/ipsec.secrets matches Azure PSK

Error: "establishing IKE_SA failed"
Fix  : Wrong Azure VPN GW IP or UDP 500 blocked

Error: "no path found"
Fix  : Check rightsubnet matches actual Azure VNet ranges

Error: "CHILD_SA closed"
Fix  : Encryption mismatch → verify ike= and esp= parameters
```

---

## 11. Security Best Practices

### 11.1 Network Security

```
✅ Zero Trust: No public IPs on VMs (Bastion only)
✅ Defense in Depth: NSG → Firewall → VPN encryption
✅ Least Privilege: NSG rules allow only required ports
✅ Traffic Inspection: All traffic through Azure Firewall
✅ Encrypted Transit: IPsec tunnel with AES-256
✅ No Split Tunneling: All traffic routed through hub
```

### 11.2 VPN Security

```
✅ Use IKEv2 (more secure than IKEv1)
✅ Use AES-256 encryption
✅ Use strong PSK (minimum 32 characters)
✅ Enable DPD (Dead Peer Detection)
✅ Rotate PSK periodically
✅ Consider certificate-based auth for production
```

### 11.3 PSK Guidelines

```
❌ Bad PSK  : password123
❌ Bad PSK  : azure-vpn
✅ Good PSK : Xk9#mP2$vN7@qL4!wR6&tY3*
✅ Better   : Use certificate-based authentication for production

Generate strong PSK:
openssl rand -base64 32
```

---

## 12. Architecture Summary

### 12.1 Final Architecture Overview

```
╔══════════════════════════════════════════════════════════════════════╗
║                    COMPLETE HYBRID ARCHITECTURE                      ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  INTERNET / ADMIN                                                    ║
║       │                                                              ║
║       ├──[HTTPS]──► Azure Bastion ──► vm-vn3l2 / vm-vnet3          ║
║       │                                                              ║
║  ┌────┴────────────────────────────────────────────────────────┐    ║
║  │                   AZURE CLOUD                               │    ║
║  │                                                             │    ║
║  │  ┌─────────────────────── HUB (vnet1) ─────────────────┐  │    ║
║  │  │                                                      │  │    ║
║  │  │   [Azure Bastion]──────────────────────────────┐    │  │    ║
║  │  │   [azure-bastion-ip]                           │    │  │    ║
║  │  │                                                │    │  │    ║
║  │  │   [Azure Firewall] ◄── All spoke traffic      │    │  │    ║
║  │  │   [azure-firewall-ip]                         │    │  │    ║
║  │  │        │                                      │    │  │    ║
║  │  │   [VPN Gateway] ◄──────────────── VPN Tunnel  │    │  │    ║
║  │  │   [vng-ip]              │                     │    │  │    ║
║  │  └────────┬────────────────│─────────────────────┘    │  │    ║
║  │           │                │                          │  │    ║
║  │  ┌────────┴──────┐ ┌───────┴───────┐                 │  │    ║
║  │  │  SPOKE1       │ │  SPOKE2       │                 │  │    ║
║  │  │  (vnet2)      │ │  (vnet3)      │                 │  │    ║
║  │  │               │ │               │◄────────────────┘  │    ║
║  │  │  [vm-vn3l2]   │ │  [vm-vnet3]   │                    │    ║
║  │  │  [NSG]        │ │  [NSG]        │                    │    ║
║  │  │  [route2]     │ │  [route3]     │                    │    ║
║  │  └───────────────┘ └───────────────┘                    │    ║
║  └─────────────────────────────────────────────────────────┘    ║
║                          │                                       ║
║              [IPsec/IKEv2 Encrypted Tunnel]                      ║
║              [AES-256 / UDP 500 + 4500]                          ║
║                          │                                       ║
║  ┌───────────────────────┴─────────────────────────────────┐    ║
║  │               ON-PREMISES (VMware)                      │    ║
║  │                                                         │    ║
║  │   ┌───────────────────────────────────────────────┐    │    ║
║  │   │          Ubuntu Router VM                     │    │    ║
║  │   │                                               │    │    ║
║  │   │  ens33 (WAN)          ens37 (LAN)             │    │    ║
║  │   │  DHCP/NAT             172.16.0.1/24           │    │    ║
║  │   │  StrongSwan ──────────────────                │    │    ║
║  │   │  IP Forward                                   │    │    ║
║  │   │  iptables NAT                                 │    │    ║
║  │   └───────────────────────────────────────────────┘    │    ║
║  │                          │                              │    ║
║  │              172.16.0.0/24 LAN                          │    ║
║  │           [On-Premises Clients]                         │    ║
║  └─────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

### 12.2 IP Address Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    IP ADDRESS REFERENCE                         │
├──────────────────────────┬──────────────────────────────────────┤
│ Component                │ IP Address / Range                   │
├──────────────────────────┼──────────────────────────────────────┤
│ vnet1 (Hub)              │ 10.1.0.0/16                         │
│ GatewaySubnet            │ 10.1.0.0/27                         │
│ AzureFirewallSubnet      │ 10.1.1.0/26                         │
│ AzureBastionSubnet       │ 10.1.2.0/27                         │
│ Azure Firewall Private   │ 10.1.1.4                            │
├──────────────────────────┼──────────────────────────────────────┤
│ vnet2 (Spoke1)           │ 10.2.0.0/16                         │
│ workload-subnet          │ 10.2.1.0/24                         │
│ vm-vn3l2                 │ 10.2.1.x (DHCP)                     │
├──────────────────────────┼──────────────────────────────────────┤
│ vnet3 (Spoke2)           │ 10.3.0.0/16                         │
│ onprem-bridge-subnet     │ 10.3.1.0/24                         │
│ vm-vnet3                 │ 10.3.1.x (DHCP)                     │
├──────────────────────────┼──────────────────────────────────────┤
│ On-Premises LAN          │ 172.16.0.0/24                       │
│ Ubuntu Router (LAN)      │ 172.16.0.1                          │
│ Ubuntu Router (WAN)      │ DHCP (VMware NAT)                   │
│ Public IP (VPN)          │ Dynamic (curl ifconfig.me)           │
├──────────────────────────┼──────────────────────────────────────┤
│ Azure VPN GW Public IP   │ From Azure Portal (vng-ip)          │
│ Azure Bastion Public IP  │ From Azure Portal (azure-bastion-ip)│
│ Azure Firewall Public IP │ From Azure Portal (azure-firewall-ip│
└──────────────────────────┴──────────────────────────────────────┘
```

---

### 12.3 Port & Protocol Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  PORT & PROTOCOL REFERENCE                      │
├──────────────┬──────────┬───────────────────────────────────────┤
│ Service      │ Protocol │ Purpose                               │
├──────────────┼──────────┼───────────────────────────────────────┤
│ IKEv2        │ UDP 500  │ VPN key exchange                      │
│ NAT-T        │ UDP 4500 │ VPN through NAT                       │
│ ESP          │ IP Proto │ Encrypted VPN data                    │
│              │ 50       │                                       │
├──────────────┼──────────┼───────────────────────────────────────┤
│ SSH          │ TCP 22   │ Linux VM management (via Bastion)     │
│ RDP          │ TCP 3389 │ Windows VM management (via Bastion)   │
│ HTTPS        │ TCP 443  │ Azure Bastion portal access           │
├──────────────┼──────────┼───────────────────────────────────────┤
│ ICMP         │ ICMP     │ Ping / connectivity testing           │
│ DNS          │ UDP 53   │ Name resolution                       │
│ HTTP/HTTPS   │ TCP 80   │ Application traffic                   │
│              │ TCP 443  │                                       │
└──────────────┴──────────┴───────────────────────────────────────┘
```

---

### 12.4 Key Design Principles Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    DESIGN PRINCIPLES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. ZERO TRUST                                                  │
│     → No direct public IP on any VM                            │
│     → All access through Azure Bastion                         │
│     → Authenticated and encrypted always                        │
│                                                                 │
│  2. DEFENSE IN DEPTH                                            │
│     → Layer 1: NSG (per VM traffic control)                    │
│     → Layer 2: Azure Firewall (central inspection)             │
│     → Layer 3: IPsec VPN (encrypted hybrid connectivity)       │
│                                                                 │
│  3. CENTRALIZED SECURITY                                        │
│     → All inter-spoke traffic through hub firewall             │
│     → Single point of policy enforcement                        │
│     → Centralized logging and monitoring                        │
│                                                                 │
│  4. NETWORK SEGMENTATION                                        │
│     → Separate VNets per workload tier                         │
│     → Controlled peering with route tables                      │
│     → On-premises isolated with VPN only                       │
│                                                                 │
│  5. ENCRYPTED CONNECTIVITY                                      │
│     → AES-256 IPsec tunnel                                     │
│     → IKEv2 for stronger authentication                         │
│     → No unencrypted hybrid traffic                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

> ## ⚠️ Important Notes
>
> - **VPN Gateway Deployment** takes 30-45 minutes in Azure
> - **Public IP Changes** on your ISP will break VPN - update Local Network Gateway
> - **PSK Must Match Exactly** on both Azure and StrongSwan sides
> - **rightsubnet** must cover all Azure VNet ranges you want to reach
> - **VMware NAT** means your Ubuntu VM is behind NAT - IKEv2 NAT-T handles this
> - **Production Use** should consider certificate-based VPN auth instead of PSK

---

*End of Documentation*
*Version 1.0 | Azure Hybrid Cloud Hub-Spoke Architecture*
```