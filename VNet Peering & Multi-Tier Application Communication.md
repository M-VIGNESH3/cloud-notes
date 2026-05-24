# Cloud Networking Fundamentals: Complete Documentation

## VNet Peering & Multi-Tier Application Communication

---

# Table of Contents

1. [OSI Model Fundamentals](#osi-model)
2. [Layer 3 and Layer 4 Communication](#layer-3-4)
3. [VLAN Concepts](#vlan)
4. [Routing Fundamentals](#routing)
5. [Network Security Groups (NSGs)](#nsgs)
6. [Virtual Networks (VNets)](#vnets)
7. [VNet Peering](#vnet-peering)
8. [Transitive Peering](#transitive-peering)
9. [2-Tier Application Architecture](#2-tier)
10. [3-Tier Application Architecture](#3-tier)
11. [Implementation Process](#implementation)

---

<a name="osi-model"></a>

# Section 1: OSI Model Fundamentals

## 1.1 What is the OSI Model?

The **Open Systems Interconnection (OSI) model** is a conceptual framework that standardizes how different network systems communicate with each other. It divides network communication into **seven distinct layers**, each with specific responsibilities.

Think of the OSI model as a postal system — each layer handles a specific part of getting your message from sender to receiver, and each layer only communicates with the layers directly above and below it.

```
┌─────────────────────────────────────────────────────────────┐
│                     OSI MODEL LAYERS                        │
├────────┬─────────────┬──────────────────────────────────────┤
│ Layer  │    Name     │           Responsibility             │
├────────┼─────────────┼──────────────────────────────────────┤
│   7    │ Application │ End-user interface, HTTP, FTP, DNS   │
├────────┼─────────────┼──────────────────────────────────────┤
│   6    │Presentation │ Data formatting, encryption, SSL/TLS │
├────────┼─────────────┼──────────────────────────────────────┤
│   5    │   Session   │ Session management, authentication   │
├────────┼─────────────┼──────────────────────────────────────┤
│   4    │  Transport  │ End-to-end delivery, TCP/UDP, ports  │
├────────┼─────────────┼──────────────────────────────────────┤
│   3    │   Network   │ Logical addressing, routing, IP      │
├────────┼─────────────┼──────────────────────────────────────┤
│   2    │  Data Link  │ MAC addressing, frames, switches     │
├────────┼─────────────┼──────────────────────────────────────┤
│   1    │  Physical   │ Cables, signals, bits, hardware      │
└────────┴─────────────┴──────────────────────────────────────┘
```

## 1.2 How Data Flows Through OSI Layers

When you send data, it travels **down** through the layers on the sender's side (encapsulation) and **up** through the layers on the receiver's side (decapsulation).

```
SENDER                                          RECEIVER
─────────────────────────────────────────────────────────────

Application  [Data]                              [Data]
                ↓                                  ↑
Presentation [Data + Formatting]      [Data + Formatting]
                ↓                                  ↑
Session      [Data + Session Info]  [Data + Session Info]
                ↓                                  ↑
Transport    [Segment: Data + Port]  [Segment: Data + Port]
                ↓                                  ↑
Network      [Packet: Seg + IP Hdr]  [Packet: Seg + IP Hdr]
                ↓                                  ↑
Data Link    [Frame: Pkt + MAC Hdr]  [Frame: Pkt + MAC Hdr]
                ↓                                  ↑
Physical     [Bits: 0101010110...]   [Bits: 0101010110...]
                ↓         ─────────────────────────↑
                └──────────────── Network ──────────┘
```

## 1.3 Data Units at Each Layer

```
┌─────────────────────────────────────────────┐
│  Layer 7 - Application  │    Data / Message │
│  Layer 6 - Presentation │    Data           │
│  Layer 5 - Session      │    Data           │
│  Layer 4 - Transport    │    Segment        │
│  Layer 3 - Network      │    Packet         │
│  Layer 2 - Data Link    │    Frame          │
│  Layer 1 - Physical     │    Bits           │
└─────────────────────────────────────────────┘
```

## 1.4 Why OSI Model Matters in Cloud Networking

| Cloud Concept | OSI Layer | Relevance |
|---------------|-----------|-----------|
| Virtual Networks (VNet) | Layer 3 | IP-based logical segmentation |
| NSG Rules | Layer 3 & 4 | IP addresses and port filtering |
| VNet Peering | Layer 3 | Routing between networks |
| Load Balancers | Layer 4 & 7 | Port and application routing |
| VLANs | Layer 2 | Network segmentation |
| SSL/TLS Certificates | Layer 6 | Encryption |

---

<a name="layer-3-4"></a>

# Section 2: Layer 3 and Layer 4 Communication

## 2.1 Layer 3 — The Network Layer

### What is Layer 3?

Layer 3 is responsible for **logical addressing** and **routing** — it determines how packets move from one network to another across multiple hops.

### Key Components of Layer 3

#### IP Addressing
Every device on a network receives a unique **IP address** that identifies it logically.

```
IPv4 Address Structure:
─────────────────────────────────────────────────────

192.168.10.50 / 24

│   │   │  │    │
│   │   │  │    └── Prefix Length (Subnet Mask)
│   │   │  └─────── Host Portion (identifies device)
└───┴───┴────────── Network Portion (identifies network)

Binary Representation:
11000000.10101000.00001010.00110010
│─ Network (24 bits) ──────────────│─ Host (8 bits) ─│
```

#### Subnet Masks

```
┌─────────────────────────────────────────────────────────┐
│              Common Subnet Masks                        │
├──────────────┬─────────────────┬────────┬──────────────┤
│ CIDR         │ Subnet Mask     │ Hosts  │ Use Case     │
├──────────────┼─────────────────┼────────┼──────────────┤
│ /8           │ 255.0.0.0       │ 16M+   │ Large Corp   │
│ /16          │ 255.255.0.0     │ 65,534 │ Large VNet   │
│ /24          │ 255.255.255.0   │ 254    │ Subnet       │
│ /26          │ 255.255.255.192 │ 62     │ Small Subnet │
│ /28          │ 255.255.255.240 │ 14     │ Tiny Subnet  │
│ /32          │ 255.255.255.255 │ 1      │ Single Host  │
└──────────────┴─────────────────┴────────┴──────────────┘
```

#### How Layer 3 Routing Works

```
Source: 10.1.0.5  ──→  Destination: 10.2.0.10

Step 1: Source checks if destination is on same network
        10.1.0.5/24  →  Network: 10.1.0.0
        10.2.0.10/24 →  Network: 10.2.0.0
        Different networks! → Send to Default Gateway

Step 2: Default Gateway (Router) checks routing table
        ┌─────────────────────────────────────────────┐
        │           ROUTING TABLE                     │
        ├─────────────────┬──────────────────────────┤
        │ Destination     │ Next Hop                 │
        ├─────────────────┼──────────────────────────┤
        │ 10.1.0.0/24     │ Directly Connected       │
        │ 10.2.0.0/24     │ 10.0.0.1 (Next Router)  │
        │ 0.0.0.0/0       │ Internet Gateway         │
        └─────────────────┴──────────────────────────┘

Step 3: Packet forwarded to next hop until destination
        is reached
```

## 2.2 Layer 4 — The Transport Layer

### What is Layer 4?

Layer 4 is responsible for **end-to-end communication** between applications. It uses **port numbers** to identify specific applications and services on a device.

### TCP vs UDP

```
┌─────────────────────────────────────────────────────────────┐
│                    TCP vs UDP Comparison                    │
├───────────────────────┬─────────────────────────────────────┤
│         TCP           │            UDP                      │
├───────────────────────┼─────────────────────────────────────┤
│ Connection-oriented   │ Connectionless                      │
│ Reliable delivery     │ No guaranteed delivery              │
│ Ordered packets       │ No ordering                         │
│ Error checking        │ Minimal error checking              │
│ Slower (overhead)     │ Faster (low overhead)               │
│ HTTP, HTTPS, SSH, FTP │ DNS, VoIP, Video Streaming          │
└───────────────────────┴─────────────────────────────────────┘
```

### TCP Three-Way Handshake

```
CLIENT                                          SERVER
  │                                               │
  │─────────── SYN (Seq=100) ──────────────────→ │
  │            "I want to connect"               │
  │                                               │
  │ ←────── SYN-ACK (Seq=200, Ack=101) ───────── │
  │         "OK, I acknowledge your request"     │
  │                                               │
  │─────────── ACK (Ack=201) ──────────────────→ │
  │            "Connection established"          │
  │                                               │
  │════════ DATA TRANSFER BEGINS ════════════════│
```

### Port Numbers

```
┌────────────────────────────────────────────────────────┐
│               Well-Known Port Numbers                  │
├──────────┬──────────┬─────────────────────────────────┤
│  Port    │ Protocol │ Service                         │
├──────────┼──────────┼─────────────────────────────────┤
│   22     │  TCP     │ SSH (Secure Shell)              │
│   80     │  TCP     │ HTTP (Web Traffic)              │
│   443    │  TCP     │ HTTPS (Secure Web)              │
│   3389   │  TCP     │ RDP (Remote Desktop)            │
│   1433   │  TCP     │ SQL Server                      │
│   3306   │  TCP     │ MySQL                           │
│   5432   │  TCP     │ PostgreSQL                      │
│   6379   │  TCP     │ Redis                           │
│   53     │  UDP     │ DNS                             │
│   8080   │  TCP     │ HTTP Alternate                  │
└──────────┴──────────┴─────────────────────────────────┘
```

### Layer 4 in Cloud Context (NSG Rules)

```
NSG Rule Example:
─────────────────────────────────────────────────────

Rule: Allow HTTPS Traffic

  Source IP:      0.0.0.0/0      (Any)
  Source Port:    *              (Any)
  Destination IP: 10.1.0.4       (Web Server)
  Destination Port: 443          (HTTPS - Layer 4)
  Protocol:       TCP            (Layer 4)
  Action:         Allow

This rule operates at Layer 3 (IP) and Layer 4 (Port)
```

---

<a name="vlan"></a>

# Section 3: VLAN Concepts

## 3.1 What is a VLAN?

A **Virtual Local Area Network (VLAN)** is a logical grouping of network devices that operates as if they are on the same physical network, even if they are physically spread across different locations.

VLANs operate at **Layer 2 (Data Link Layer)** and use **VLAN tags (802.1Q)** to identify which VLAN a frame belongs to.

## 3.2 Why VLANs are Used

```
WITHOUT VLANs (Flat Network):
─────────────────────────────────────────────────────────────

[HR PC] ─────────────────────────── [Finance PC]
           │                   │
        [Switch]            [Server]
           │
        [IT PC] ─── Everyone can see everyone's traffic!
                    Security risk, performance issues

─────────────────────────────────────────────────────────────

WITH VLANs (Segmented Network):
─────────────────────────────────────────────────────────────

VLAN 10 (HR):      [HR-PC1] ──── [HR-PC2]
                        │
                    [Managed Switch] ── VLAN Tags separate traffic
                        │
VLAN 20 (Finance): [Fin-PC1] ── [Fin-PC2]
                        │
VLAN 30 (IT):      [IT-PC1] ─── [IT-PC2]

HR cannot communicate with Finance without going through Router!
```

## 3.3 How VLAN Tagging Works (802.1Q)

```
Standard Ethernet Frame:
┌──────────┬─────────┬──────────┬──────────┐
│Dest MAC  │Src MAC  │ EtherType│  Data    │
│(6 bytes) │(6 bytes)│(2 bytes) │          │
└──────────┴─────────┴──────────┴──────────┘

802.1Q Tagged Frame:
┌──────────┬─────────┬────────────────┬──────────┬──────────┐
│Dest MAC  │Src MAC  │ 802.1Q Tag     │EtherType │  Data    │
│(6 bytes) │(6 bytes)│(4 bytes)       │(2 bytes) │          │
└──────────┴─────────┴────────────────┴──────────┴──────────┘
                      │
                      ▼
               ┌──────────────┐
               │  VLAN Tag    │
               ├──────────────┤
               │ TPID: 0x8100 │ ← Identifies as 802.1Q
               │ PCP: 3 bits  │ ← Priority
               │ DEI: 1 bit   │ ← Drop Eligible
               │ VID: 12 bits │ ← VLAN ID (1-4094)
               └──────────────┘
```

## 3.4 VLANs in Cloud Computing

In cloud environments like **Azure**, the concept of VLANs translates into **Subnets within Virtual Networks (VNets)**:

```
Traditional Network          Cloud Equivalent
────────────────────         ─────────────────────────────

Physical Network     ─────→  Virtual Network (VNet)
VLAN                 ─────→  Subnet
VLAN ID              ─────→  Subnet CIDR Range
Inter-VLAN Routing   ─────→  VNet Routing / Peering
VLAN ACLs            ─────→  Network Security Groups (NSGs)
```

### Azure VNet with Subnets (Equivalent to VLANs)

```
Azure Virtual Network: 10.0.0.0/16
│
├── Subnet: Web-Tier    (10.0.1.0/24)  → Like VLAN 10
│       │
│       └── Web Servers (10.0.1.4, 10.0.1.5)
│
├── Subnet: App-Tier    (10.0.2.0/24)  → Like VLAN 20
│       │
│       └── App Servers (10.0.2.4, 10.0.2.5)
│
└── Subnet: DB-Tier     (10.0.3.0/24)  → Like VLAN 30
        │
        └── DB Servers  (10.0.3.4, 10.0.3.5)
```

---

<a name="routing"></a>

# Section 4: Routing Fundamentals

## 4.1 What is Routing?

**Routing** is the process of selecting paths in a network along which to send network traffic. A **router** examines each incoming packet's destination IP address and forwards it to the appropriate next-hop location.

## 4.2 Types of Routing

```
┌─────────────────────────────────────────────────────────────┐
│                    Types of Routing                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  STATIC ROUTING                                             │
│  ─────────────                                              │
│  • Manually configured routes                               │
│  • No automatic updates                                     │
│  • Simple, predictable                                      │
│  • Good for small/stable networks                           │
│                                                             │
│  DYNAMIC ROUTING                                            │
│  ───────────────                                            │
│  • Routes learned automatically via protocols               │
│  • Protocols: BGP, OSPF, RIP                               │
│  • Adapts to network changes                                │
│  • Used in large, complex networks                          │
│                                                             │
│  SYSTEM ROUTES (Azure)                                      │
│  ─────────────────────                                      │
│  • Automatically created by Azure                           │
│  • Handles VNet, Subnet, VNet Peering traffic              │
│  • Cannot be deleted but can be overridden                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 4.3 Routing Table

A routing table is a database of routes that tells the router where to send packets.

```
ROUTING TABLE EXAMPLE:
─────────────────────────────────────────────────────────────

┌──────────────────┬─────────────────┬──────────┬──────────┐
│  Destination     │  Subnet Mask    │ Next Hop │ Metric   │
├──────────────────┼─────────────────┼──────────┼──────────┤
│  10.1.0.0        │  255.255.255.0  │ On-Link  │   1      │
│  10.2.0.0        │  255.255.255.0  │ 10.1.0.1 │  10      │
│  10.3.0.0        │  255.255.255.0  │ 10.1.0.1 │  10      │
│  0.0.0.0         │  0.0.0.0        │ 10.1.0.1 │   1      │
└──────────────────┴─────────────────┴──────────┴──────────┘

Reading the table:
• 10.1.0.0/24  → Directly connected (On-Link)
• 10.2.0.0/24  → Send to 10.1.0.1 (next hop router)
• 10.3.0.0/24  → Send to 10.1.0.1 (next hop router)
• 0.0.0.0/0    → Default route (everything else)
```

## 4.4 Azure System Routes

Azure automatically creates system routes for each subnet:

```
Azure Auto-Created Routes:
─────────────────────────────────────────────────────────────

VNet: 10.0.0.0/16
Subnet: 10.0.1.0/24

┌──────────────────┬──────────────────────────────────────┐
│  Address Prefix  │  Next Hop Type                       │
├──────────────────┼──────────────────────────────────────┤
│  10.0.0.0/16     │  Virtual Network (VNet local)        │
│  0.0.0.0/0       │  Internet                            │
│  10.0.0.0/8      │  None (RFC 1918 - blocked)           │
│  172.16.0.0/12   │  None (RFC 1918 - blocked)           │
│  192.168.0.0/16  │  None (RFC 1918 - blocked)           │
└──────────────────┴──────────────────────────────────────┘

When VNet Peering is added:
┌──────────────────┬──────────────────────────────────────┐
│  10.2.0.0/16     │  VNet Peering (peered network)       │
└──────────────────┴──────────────────────────────────────┘
```

## 4.5 User-Defined Routes (UDR)

UDRs allow you to override Azure's default system routes:

```
Use Case: Force traffic through a Network Virtual Appliance (NVA)

Without UDR:                        With UDR:
─────────────                        ──────────────────────
Web VM                               Web VM
  │                                    │
  │ Direct route                       │ Custom route
  ↓                                    ↓
DB VM                                Firewall NVA
(No inspection)                        │
                                       │ Inspect & forward
                                       ↓
                                    DB VM
                                  (Traffic inspected)

UDR Configuration:
  Address Prefix: 10.0.3.0/24 (DB Subnet)
  Next Hop Type:  Virtual Appliance
  Next Hop IP:    10.0.4.4 (Firewall IP)
```

---

<a name="nsgs"></a>

# Section 5: Network Security Groups (NSGs)

## 5.1 What is an NSG?

A **Network Security Group (NSG)** is a cloud firewall that contains a list of security rules that **allow or deny** inbound and outbound network traffic based on:

- Source IP address / CIDR range
- Destination IP address / CIDR range
- Protocol (TCP, UDP, Any)
- Port numbers (source and destination)
- Direction (Inbound or Outbound)

## 5.2 NSG Architecture

```
NSG can be associated with:
─────────────────────────────────────────────────────────────

1. SUBNET LEVEL (Applies to all resources in subnet)
   ┌─────────────────────────────────┐
   │  Subnet: 10.0.1.0/24           │
   │  ┌──────────┐  ┌──────────┐    │  ← NSG applied here
   │  │  VM-1    │  │  VM-2    │    │
   │  └──────────┘  └──────────┘    │
   └─────────────────────────────────┘

2. NIC (NETWORK INTERFACE CARD) LEVEL (Applies to single VM)
   ┌─────────────────────────────────┐
   │  Subnet: 10.0.1.0/24           │
   │  ┌──────────┐  ┌──────────┐    │
   │  │  VM-1    │  │  VM-2    │    │  ← NSG applied to VM-1 NIC
   │  └────┬─────┘  └──────────┘    │
   │       │ NSG                    │
   └───────┼─────────────────────────┘
           ↓
```

## 5.3 NSG Rule Structure

```
NSG RULE COMPONENTS:
─────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────┐
│                    NSG Rule                             │
├────────────────────────┬────────────────────────────────┤
│  Priority              │ 100 - 4096 (lower = higher     │
│                        │ priority, processed first)     │
├────────────────────────┼────────────────────────────────┤
│  Name                  │ Descriptive name               │
├────────────────────────┼────────────────────────────────┤
│  Direction             │ Inbound or Outbound            │
├────────────────────────┼────────────────────────────────┤
│  Source                │ IP, CIDR, Service Tag, ASG     │
├────────────────────────┼────────────────────────────────┤
│  Source Port Range     │ *, single port, or range       │
├────────────────────────┼────────────────────────────────┤
│  Destination           │ IP, CIDR, Service Tag, ASG     │
├────────────────────────┼────────────────────────────────┤
│  Destination Port      │ *, single port, or range       │
├────────────────────────┼────────────────────────────────┤
│  Protocol              │ TCP, UDP, ICMP, Any            │
├────────────────────────┼────────────────────────────────┤
│  Action                │ Allow or Deny                  │
└────────────────────────┴────────────────────────────────┘
```

## 5.4 Default NSG Rules

Azure includes default rules that cannot be deleted:

```
DEFAULT INBOUND RULES:
┌──────────┬──────────────────┬───────────────┬──────┬────────┐
│ Priority │ Name             │ Source        │ Port │ Action │
├──────────┼──────────────────┼───────────────┼──────┼────────┤
│  65000   │ AllowVnetInBound │ VirtualNetwork│  *   │ Allow  │
│  65001   │ AllowAzureLBIn   │ AzureLoadBal  │  *   │ Allow  │
│  65500   │ DenyAllInBound   │ *             │  *   │ Deny   │
└──────────┴──────────────────┴───────────────┴──────┴────────┘

DEFAULT OUTBOUND RULES:
┌──────────┬───────────────────┬────────┬──────┬────────┐
│ Priority │ Name              │ Dest   │ Port │ Action │
├──────────┼───────────────────┼────────┼──────┼────────┤
│  65000   │ AllowVnetOutBound │ VNet   │  *   │ Allow  │
│  65001   │ AllowInternetOut  │ Inet   │  *   │ Allow  │
│  65500   │ DenyAllOutBound   │ *      │  *   │ Deny   │
└──────────┴───────────────────┴────────┴──────┴────────┘
```

## 5.5 NSG Traffic Flow Processing

```
INBOUND TRAFFIC PROCESSING:
─────────────────────────────────────────────────────────────

Internet → Subnet NSG → NIC NSG → Virtual Machine

Step 1: Packet arrives at Azure border
Step 2: Subnet-level NSG rules evaluated (lowest priority first)
Step 3: If allowed, NIC-level NSG rules evaluated
Step 4: If allowed, packet reaches the VM

OUTBOUND TRAFFIC PROCESSING:
─────────────────────────────────────────────────────────────

Virtual Machine → NIC NSG → Subnet NSG → Internet

Step 1: Packet leaves VM
Step 2: NIC-level NSG rules evaluated
Step 3: If allowed, Subnet-level NSG rules evaluated
Step 4: If allowed, packet exits to destination
```

## 5.6 NSG Service Tags

Service Tags are predefined identifiers for Azure services, simplifying NSG configuration:

```
┌──────────────────────┬──────────────────────────────────┐
│  Service Tag         │  Represents                      │
├──────────────────────┼──────────────────────────────────┤
│  Internet            │  Public Internet space           │
│  VirtualNetwork      │  All VNet address spaces         │
│  AzureLoadBalancer   │  Azure Load Balancer IPs         │
│  AzureCloud          │  All Azure datacenter IPs        │
│  Storage             │  Azure Storage service IPs       │
│  Sql                 │  Azure SQL service IPs           │
│  AppService          │  Azure App Service IPs           │
└──────────────────────┴──────────────────────────────────┘

Usage in NSG Rule:
Source: Internet  → Destination: 10.0.1.0/24  Port: 443  ALLOW
(Instead of specifying all internet IPs)
```

## 5.7 NSG for 3-Tier Application

```
3-TIER NSG DESIGN:
─────────────────────────────────────────────────────────────

WEB TIER NSG (Subnet: 10.0.1.0/24):
┌────────┬────────────┬──────────────┬──────┬────────┐
│Priority│ Direction  │ Source       │ Port │ Action │
├────────┼────────────┼──────────────┼──────┼────────┤
│  100   │ Inbound    │ Internet     │ 80   │ Allow  │
│  110   │ Inbound    │ Internet     │ 443  │ Allow  │
│  200   │ Outbound   │ 10.0.1.0/24  │ 8080 │ Allow  │
│ 65500  │ Inbound    │ *            │ *    │ Deny   │
└────────┴────────────┴──────────────┴──────┴────────┘

APP TIER NSG (Subnet: 10.0.2.0/24):
┌────────┬────────────┬──────────────┬──────┬────────┐
│Priority│ Direction  │ Source       │ Port │ Action │
├────────┼────────────┼──────────────┼──────┼────────┤
│  100   │ Inbound    │ 10.0.1.0/24  │ 8080 │ Allow  │
│  200   │ Outbound   │ 10.0.2.0/24  │ 1433 │ Allow  │
│ 65500  │ Inbound    │ *            │ *    │ Deny   │
└────────┴────────────┴──────────────┴──────┴────────┘

DB TIER NSG (Subnet: 10.0.3.0/24):
┌────────┬────────────┬──────────────┬──────┬────────┐
│Priority│ Direction  │ Source       │ Port │ Action │
├────────┼────────────┼──────────────┼──────┼────────┤
│  100   │ Inbound    │ 10.0.2.0/24  │ 1433 │ Allow  │
│ 65500  │ Inbound    │ *            │ *    │ Deny   │
└────────┴────────────┴──────────────┴──────┴────────┘
```

---

<a name="vnets"></a>

# Section 6: Virtual Networks (VNets)

## 6.1 What is a Virtual Network?

A **Virtual Network (VNet)** is a logically isolated network in the cloud that closely resembles a traditional network you'd operate in your own data center. It enables Azure resources to securely communicate with each other, the internet, and on-premises networks.

## 6.2 VNet Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure Virtual Network                    │
│                    10.0.0.0/16                              │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Subnet: Web-Tier (10.0.1.0/24)                     │  │
│  │  ┌─────────────┐  ┌─────────────┐                   │  │
│  │  │  Web VM 1   │  │  Web VM 2   │                   │  │
│  │  │ 10.0.1.4    │  │ 10.0.1.5    │                   │  │
│  │  └─────────────┘  └─────────────┘                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Subnet: App-Tier (10.0.2.0/24)                     │  │
│  │  ┌─────────────┐  ┌─────────────┐                   │  │
│  │  │  App VM 1   │  │  App VM 2   │                   │  │
│  │  │ 10.0.2.4    │  │ 10.0.2.5    │                   │  │
│  │  └─────────────┘  └─────────────┘                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Subnet: DB-Tier (10.0.3.0/24)                      │  │
│  │  ┌─────────────┐                                    │  │
│  │  │  DB Server  │                                    │  │
│  │  │ 10.0.3.4    │                                    │  │
│  │  └─────────────┘                                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 6.3 VNet Address Space Planning

```
ADDRESS SPACE PLANNING GUIDELINES:
─────────────────────────────────────────────────────────────

RFC 1918 Private Address Ranges (Recommended for VNets):
┌─────────────────┬─────────────────────────────────────────┐
│  Range          │  Size                                   │
├─────────────────┼─────────────────────────────────────────┤
│  10.0.0.0/8     │  16,777,216 addresses                   │
│  172.16.0.0/12  │  1,048,576 addresses                    │
│  192.168.0.0/16 │  65,536 addresses                       │
└─────────────────┴─────────────────────────────────────────┘

AZURE RESERVED ADDRESSES IN EACH SUBNET:
Azure reserves 5 IP addresses in every subnet:

Subnet: 10.0.1.0/24
├── 10.0.1.0   → Network Address
├── 10.0.1.1   → Default Gateway (Azure)
├── 10.0.1.2   → Azure DNS
├── 10.0.1.3   → Azure Reserved
└── 10.0.1.255 → Broadcast Address

Usable addresses: 10.0.1.4 → 10.0.1.254 = 251 addresses
```

## 6.4 VNet Key Properties

```
┌─────────────────────────────────────────────────────────────┐
│                   VNet Key Properties                       │
├────────────────────────┬────────────────────────────────────┤
│  Property              │  Description                       │
├────────────────────────┼────────────────────────────────────┤
│  Region-Specific       │  VNet exists in one Azure region   │
│  Subscription-Scoped   │  Belongs to one subscription       │
│  Multiple CIDR Blocks  │  Can have multiple address spaces  │
│  DNS Configuration     │  Custom or Azure-provided DNS      │
│  DDoS Protection       │  Basic (free) or Standard (paid)  │
│  No Cross-Region       │  Cannot span multiple regions      │
└────────────────────────┴────────────────────────────────────┘
```

---

<a name="vnet-peering"></a>

# Section 7: VNet Peering

## 7.1 What is VNet Peering?

**VNet Peering** connects two Azure virtual networks, enabling resources in either virtual network to communicate with each other. The traffic between peered VNets travels through Microsoft's private backbone network — **not** through the public internet.

## 7.2 Types of VNet Peering

```
┌─────────────────────────────────────────────────────────────┐
│                  Types of VNet Peering                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. VIRTUAL NETWORK PEERING                                 │
│     ─────────────────────                                   │
│     • Connects VNets within the SAME region                 │
│     • Ultra-low latency                                     │
│     • High bandwidth                                        │
│     • Most common type                                      │
│                                                             │
│  2. GLOBAL VIRTUAL NETWORK PEERING                          │
│     ──────────────────────────────                          │
│     • Connects VNets in DIFFERENT regions                   │
│     • Slightly higher latency (cross-region)                │
│     • Traffic stays on Microsoft backbone                   │
│     • Useful for DR and multi-region architectures          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 7.3 VNet Peering Architecture

```
VNet Peering Connection:
─────────────────────────────────────────────────────────────

VNet-A (10.1.0.0/16)          VNet-B (10.2.0.0/16)
┌─────────────────┐           ┌─────────────────┐
│                 │           │                 │
│  Web Servers    │           │  Shared         │
│  10.1.1.0/24   │◄─────────►│  Services       │
│                 │  Peering  │  10.2.1.0/24   │
│  App Servers    │  Link     │                 │
│  10.1.2.0/24   │           │  Monitoring     │
│                 │           │  10.2.2.0/24   │
└─────────────────┘           └─────────────────┘

• Both directions must be configured (bi-directional)
• Non-overlapping address spaces required
• No gateway required for peering
• Traffic stays on Azure backbone
```

## 7.4 VNet Peering Requirements

```
REQUIREMENTS FOR VNET PEERING:
─────────────────────────────────────────────────────────────

✅ REQUIREMENTS:
┌────────────────────────────────────────────────────────────┐
│  1. Non-overlapping address spaces                         │
│     VNet-A: 10.1.0.0/16 ✅                                │
│     VNet-B: 10.2.0.0/16 ✅                                │
│     (Cannot use 10.0.0.0/16 for both)                      │
│                                                            │
│  2. Peering is not transitive                              │
│     A peers B, B peers C ≠ A can reach C                  │
│                                                            │
│  3. Both peerings must be created                          │
│     A→B peering AND B→A peering required                   │
│                                                            │
│  4. Sufficient permissions                                 │
│     Contributor role on both VNets                         │
└────────────────────────────────────────────────────────────┘

❌ LIMITATIONS:
┌────────────────────────────────────────────────────────────┐
│  • Overlapping CIDR blocks not allowed                     │
│  • Cannot resize address space after peering               │
│  • Maximum 500 peerings per VNet (default)                 │
│  • Peering is NOT transitive (requires hub-spoke)          │
└────────────────────────────────────────────────────────────┘
```

## 7.5 VNet Peering Configuration Options

```
PEERING CONFIGURATION SETTINGS:
─────────────────────────────────────────────────────────────

When creating a peering link, configure:

┌────────────────────────────────────────────────────────────┐
│ SETTING                    │ OPTIONS    │ DESCRIPTION      │
├────────────────────────────┼────────────┼──────────────────┤
│ Traffic to remote VNet     │ Allow/Block│ Forward traffic  │
├────────────────────────────┼────────────┼──────────────────┤
│ Traffic from remote VNet   │ Allow/Block│ Receive traffic  │
├────────────────────────────┼────────────┼──────────────────┤
│ Gateway transit            │ On/Off     │ Use peer gateway │
├────────────────────────────┼────────────┼──────────────────┤
│ Remote gateways            │ On/Off     │ Use remote gw    │
└────────────────────────────┴────────────┴──────────────────┘

Gateway Transit Use Case:
─────────────────────────
Hub VNet has VPN/ExpressRoute gateway
Spoke VNets can use Hub's gateway via "Use Remote Gateway"
This avoids creating gateway in each spoke
```

## 7.6 How Peering Updates Routing Tables

When peering is established, Azure automatically updates routing tables:

```
BEFORE PEERING:
─────────────────────────────────────────────────────────────

VNet-A Route Table:
┌──────────────┬──────────────────────────┐
│ Destination  │ Next Hop                 │
├──────────────┼──────────────────────────┤
│ 10.1.0.0/16  │ Virtual Network          │
│ 0.0.0.0/0    │ Internet                 │
└──────────────┴──────────────────────────┘

AFTER PEERING VNet-A ↔ VNet-B:
─────────────────────────────────────────────────────────────

VNet-A Route Table (Updated Automatically):
┌──────────────┬──────────────────────────┐
│ Destination  │ Next Hop                 │
├──────────────┼──────────────────────────┤
│ 10.1.0.0/16  │ Virtual Network          │
│ 10.2.0.0/16  │ VNet Peering ← NEW      │
│ 0.0.0.0/0    │ Internet                 │
└──────────────┴──────────────────────────┘
```

---

<a name="transitive-peering"></a>

# Section 8: Transitive Peering

## 8.1 What is Transitive Peering?

**Transitive peering** refers to the concept of routing traffic through an intermediate VNet to reach another VNet that is not directly peered.

> **Critical Concept:** Azure VNet Peering is **NOT transitive by default.**

## 8.2 The Non-Transitive Problem

```
THE PROBLEM — Non-Transitive Peering:
─────────────────────────────────────────────────────────────

VNet-A ◄────── Peered ──────► VNet-B ◄────── Peered ──────► VNet-C

Question: Can VNet-A communicate with VNet-C?

ANSWER: ❌ NO — Peering is NOT transitive!

Even though:
  A peers with B ✅
  B peers with C ✅
  A CANNOT reach C directly ❌

VNet-A (10.1.0.0/16)
    │
    │ Peering
    │
VNet-B (10.2.0.0/16) ← Hub
    │
    │ Peering
    │
VNet-C (10.3.0.0/16)

VNet-A → VNet-B = ✅ (Allowed)
VNet-B → VNet-C = ✅ (Allowed)
VNet-A → VNet-C = ❌ (NOT Allowed — must add direct peering)
```

## 8.3 Solutions for Transitive Connectivity

### Solution 1: Full Mesh Peering (Small Scale)

```
FULL MESH — Direct peering between all VNets:
─────────────────────────────────────────────────────────────

       VNet-A
      /  |  \
     /   |   \
    /    |    \
VNet-B──────VNet-C

Peerings:
• A ↔ B (2 peering links)
• B ↔ C (2 peering links)
• A ↔ C (2 peering links)

Total: 6 peering links for 3 VNets

Formula: n*(n-1) peering links for n VNets
• 4 VNets = 12 links
• 10 VNets = 90 links ← Gets complex quickly!

✅ Best for: Small number of VNets (< 5)
❌ Not scalable for large environments
```

### Solution 2: Hub-Spoke with NVA (Recommended)

```
HUB-SPOKE TOPOLOGY WITH NVA:
─────────────────────────────────────────────────────────────

         ┌─────────────────────────┐
         │     HUB VNet            │
         │     10.0.0.0/16        │
         │  ┌──────────────────┐  │
         │  │  Network Virtual │  │
         │  │  Appliance (NVA) │  │
         │  │  or Azure        │  │
         │  │  Firewall        │  │
         │  └──────────────────┘  │
         └─────────────────────────┘
               /      |      \
              /       |       \
    Peering  /  Peering  Peering  \
            /         |            \
     ┌──────┐     ┌──────┐     ┌──────┐
     │Spoke │     │Spoke │     │Spoke │
     │VNet-A│     │VNet-B│     │VNet-C│
     └──────┘     └──────┘     └──────┘

Traffic Flow A → C:
VNet-A → Hub NVA → VNet-C

Requires:
1. UDR in each Spoke: Route traffic to Hub NVA
2. IP Forwarding enabled on NVA
3. NVA routes traffic to destination spoke
```

### Solution 3: Azure Virtual WAN

```
AZURE VIRTUAL WAN (Managed Hub-Spoke):
─────────────────────────────────────────────────────────────

         ┌────────────────────────────┐
         │    Azure Virtual WAN       │
         │    (Managed Hub)           │
         │                            │
         │    Auto-manages routing    │
         │    between all spokes      │
         └────────────────────────────┘
               ↙    ↓    ↓    ↘
           Spoke  Spoke  Spoke  Spoke
           VNet-A VNet-B VNet-C VNet-D

✅ Fully managed by Azure
✅ Auto-routing between spokes
✅ Scales to hundreds of VNets
✅ Built-in VPN and ExpressRoute support
```

## 8.4 UDR Configuration for Hub-Spoke

```
USER-DEFINED ROUTES FOR HUB-SPOKE:
─────────────────────────────────────────────────────────────

SPOKE-A Route Table (Applied to all Spoke-A subnets):
┌──────────────┬────────────────┬──────────────────────────┐
│ Destination  │ Next Hop Type  │ Next Hop IP              │
├──────────────┼────────────────┼──────────────────────────┤
│ 10.1.0.0/16  │ Virt Appliance │ 10.0.0.4 (Hub NVA)      │
│ 10.2.0.0/16  │ Virt Appliance │ 10.0.0.4 (Hub NVA)      │
│ 10.3.0.0/16  │ Virt Appliance │ 10.0.0.4 (Hub NVA)      │
└──────────────┴────────────────┴──────────────────────────┘

HUB NVA must have:
• IP Forwarding ENABLED on the NIC
• Routing rules to forward to correct spoke
• (Or Azure Firewall handles this automatically)
```

---

<a name="2-tier"></a>

# Section 9: 2-Tier Application Architecture

## 9.1 What is a 2-Tier Architecture?

A **2-tier architecture** (also called client-server architecture) divides an application into two layers:

1. **Presentation/Web Tier** — User-facing layer (web servers, frontend)
2. **Data Tier** — Database layer (data storage and retrieval)

## 9.2 2-Tier Architecture Overview

```
2-TIER APPLICATION ARCHITECTURE:
─────────────────────────────────────────────────────────────

Internet Users
      │
      │ HTTPS (443)
      ▼
┌─────────────────────────────────────────────────────────┐
│                  Azure Virtual Network                  │
│                  10.0.0.0/16                            │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Web/Application Tier (10.0.1.0/24)              │  │
│  │                                                   │  │
│  │   ┌──────────┐      ┌──────────┐                 │  │
│  │   │ Web VM 1 │      │ Web VM 2 │                 │  │
│  │   │10.0.1.4  │      │10.0.1.5  │                 │  │
│  │   └──────────┘      └──────────┘                 │  │
│  └───────────────────────────────────────────────────┘  │
│                         │                               │
│                    Port 1433 (SQL)                       │
│                         │                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Database Tier (10.0.2.0/24)                     │  │
│  │                                                   │  │
│  │   ┌──────────────────────────┐                   │  │
│  │   │  Database Server         │                   │  │
│  │   │  SQL / MySQL / PostgreSQL│                   │  │
│  │   │  10.0.2.4                │                   │  │
│  │   └──────────────────────────┘                   │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 9.3 2-Tier IP Address Planning

```
2-TIER IP ADDRESS PLAN:
─────────────────────────────────────────────────────────────

Virtual Network: 10.0.0.0/16

┌─────────────────────────────────────────────────────────┐
│  Tier          │  Subnet CIDR   │  VM IPs              │
├─────────────────┼────────────────┼──────────────────────┤
│  Web/App Tier  │  10.0.1.0/24  │  10.0.1.4 - .254     │
│                │               │  Web-VM1: 10.0.1.4    │
│                │               │  Web-VM2: 10.0.1.5    │
├─────────────────┼────────────────┼──────────────────────┤
│  Database Tier │  10.0.2.0/24  │  10.0.2.4 - .254     │
│                │               │  DB-Server: 10.0.2.4  │
└─────────────────┴────────────────┴──────────────────────┘
```

## 9.4 2-Tier NSG Design

```
WEB TIER NSG RULES:
─────────────────────────────────────────────────────────────

INBOUND:
┌────────┬───────────┬──────────────┬──────┬──────────┐
│ Priority│ Source    │ Destination  │ Port │ Action   │
├────────┼───────────┼──────────────┼──────┼──────────┤
│  100   │ Internet  │ 10.0.1.0/24  │  80  │ Allow    │
│  110   │ Internet  │ 10.0.1.0/24  │ 443  │ Allow    │
│  120   │ [AdminIP] │ 10.0.1.0/24  │  22  │ Allow    │
│ 65500  │ *         │ *            │  *   │ Deny     │
└────────┴───────────┴──────────────┴──────┴──────────┘

OUTBOUND:
┌────────┬───────────┬──────────────┬──────┬──────────┐
│ Priority│ Source    │ Destination  │ Port │ Action   │
├────────┼───────────┼──────────────┼──────┼──────────┤
│  100   │10.0.1.0/24│ 10.0.2.0/24  │ 1433 │ Allow    │
│  200   │10.0.1.0/24│ Internet     │ 443  │ Allow    │
│ 65500  │ *         │ *            │  *   │ Deny     │
└────────┴───────────┴──────────────┴──────┴──────────┘

DATABASE TIER NSG RULES:
─────────────────────────────────────────────────────────────

INBOUND:
┌────────┬───────────┬──────────────┬──────┬──────────┐
│ Priority│ Source    │ Destination  │ Port │ Action   │
├────────┼───────────┼──────────────┼──────┼──────────┤
│  100   │10.0.1.0/24│ 10.0.2.0/24  │ 1433 │ Allow    │
│ 65500  │ *         │ *            │  *   │ Deny     │
└────────┴───────────┴──────────────┴──────┴──────────┘

OUTBOUND:
┌────────┬───────────┬──────────────┬──────┬──────────┐
│ Priority│ Source    │ Destination  │ Port │ Action   │
├────────┼───────────┼──────────────┼──────┼──────────┤
│  100   │10.0.2.0/24│ 10.0.1.0/24  │ *    │ Allow    │
│ 65500  │ *         │ *            │  *   │ Deny     │
└────────┴───────────┴──────────────┴──────┴──────────┘
```

## 9.5 2-Tier Traffic Flow

```
2-TIER REQUEST FLOW:
─────────────────────────────────────────────────────────────

User Browser → HTTPS Request → www.example.com

Step 1: DNS resolves to Load Balancer Public IP
        User → DNS → 20.50.100.1 (Public IP)

Step 2: Load Balancer distributes to Web VM
        20.50.100.1 → 10.0.1.4 (Web-VM1)
        NSG Check: Port 443 from Internet = ALLOW ✅

Step 3: Web VM processes request, queries database
        10.0.1.4 → 10.0.2.4 (DB-Server) Port 1433
        NSG Check: Port 1433 from Web Tier = ALLOW ✅

Step 4: Database returns data to Web VM
        10.0.2.4 → 10.0.1.4 (Response)

Step 5: Web VM sends HTML response to user
        10.0.1.4 → User Browser
```

---

<a name="3-tier"></a>

# Section 10: 3-Tier Application Architecture

## 10.1 What is a 3-Tier Architecture?

A **3-tier architecture** extends the 2-tier model by separating the application logic into its own dedicated tier:

1. **Presentation Tier (Web)** — User interface, HTTP/HTTPS handling
2. **Application/Logic Tier (App)** — Business logic, API processing
3. **Data Tier (Database)** — Data persistence and retrieval

This separation provides better **security**, **scalability**, and **maintainability**.

## 10.2 3-Tier Architecture Overview

```
3-TIER APPLICATION ARCHITECTURE:
─────────────────────────────────────────────────────────────

                    INTERNET USERS
                          │
                          │ HTTPS (Port 443)
                          ▼
                  ┌───────────────┐
                  │ Public Load   │
                  │ Balancer      │
                  │ (Public IP)   │
                  └───────┬───────┘
                          │
═══════════════════════════════════════════════════════════════
                 AZURE VIRTUAL NETWORK (10.0.0.0/16)
═══════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────┐
│  TIER 1 - WEB SUBNET (10.0.1.0/24)                     │
│  NSG: Allow 443 inbound from Internet                   │
│  NSG: Allow 8080 outbound to App Tier                   │
│                                                         │
│   ┌──────────────┐      ┌──────────────┐               │
│   │  Web VM 1    │      │  Web VM 2    │               │
│   │  IIS/Nginx   │      │  IIS/Nginx   │               │
│   │  10.0.1.4    │      │  10.0.1.5    │               │
│   └──────────────┘      └──────────────┘               │
└───────────────────────────────┬─────────────────────────┘
                                │ Port 8080
                                │
┌───────────────────────────────▼─────────────────────────┐
│  TIER 2 - APP SUBNET (10.0.2.0/24)                      │
│  NSG: Allow 8080 inbound from Web Tier only             │
│  NSG: Allow 1433 outbound to DB Tier                    │
│                                                         │
│   ┌──────────────┐      ┌──────────────┐               │
│   │  App VM 1    │      │  App VM 2    │               │
│   │  Node.js/    │      │  Node.js/    │               │
│   │  Java/.NET   │      │  Java/.NET   │               │
│   │  10.0.2.4    │      │  10.0.2.5    │               │
│   └──────────────┘      └──────────────┘               │
└───────────────────────────────┬─────────────────────────┘
                                │ Port 1433
                                │
┌───────────────────────────────▼─────────────────────────┐
│  TIER 3 - DATABASE SUBNET (10.0.3.0/24)                 │
│  NSG: Allow 1433 inbound from App Tier ONLY             │
│  NSG: Deny all other inbound                            │
│                                                         │
│   ┌──────────────────────────────────────────┐         │
│   │  Database Server                         │         │
│   │  SQL Server / MySQL / PostgreSQL         │         │
│   │  10.0.3.4                                │         │
│   └──────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘
```

## 10.3 3-Tier IP Address Plan

```
3-TIER IP ADDRESS PLANNING:
─────────────────────────────────────────────────────────────

Virtual Network: 10.0.0.0/16

┌──────────────┬────────────────┬──────────┬────────────────┐
│  Tier        │  Subnet        │ Size     │ VM IPs         │
├──────────────┼────────────────┼──────────┼────────────────┤
│  Web         │ 10.0.1.0/24   │  /24     │ .4, .5, .6    │
│  Application │ 10.0.2.0/24   │  /24     │ .4, .5, .6    │
│  Database    │ 10.0.3.0/24   │  /24     │ .4 (Primary)  │
│  Management  │ 10.0.4.0/28   │  /28     │ .4 (Bastion)  │
│  Gateway     │ 10.0.5.0/27   │  /27     │ Reserved       │
└──────────────┴────────────────┴──────────┴────────────────┘

Note: GatewaySubnet MUST be named exactly "GatewaySubnet"
```

## 10.4 3-Tier NSG Design

```
WEB TIER NSG:
─────────────────────────────────────────────────────────────

INBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │ Internet    │ 10.0.1.0/24  │  80  │ Allow  │
│  110   │ Internet    │ 10.0.1.0/24  │ 443  │ Allow  │
│  120   │ AzureLB     │ 10.0.1.0/24  │  *   │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘

OUTBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │10.0.1.0/24  │ 10.0.2.0/24  │ 8080 │ Allow  │
│  200   │10.0.1.0/24  │ Internet     │ 443  │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘

APP TIER NSG:
─────────────────────────────────────────────────────────────

INBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │10.0.1.0/24  │ 10.0.2.0/24  │ 8080 │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘

OUTBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │10.0.2.0/24  │ 10.0.3.0/24  │ 1433 │ Allow  │
│  200   │10.0.2.0/24  │ Internet     │ 443  │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘

DB TIER NSG:
─────────────────────────────────────────────────────────────

INBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │10.0.2.0/24  │ 10.0.3.0/24  │ 1433 │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘

OUTBOUND RULES:
┌────────┬─────────────┬──────────────┬──────┬────────┐
│Priority│ Source      │ Destination  │ Port │ Action │
├────────┼─────────────┼──────────────┼──────┼────────┤
│  100   │10.0.3.0/24  │ 10.0.2.0/24  │  *   │ Allow  │
│ 65500  │ *           │ *            │  *   │ Deny   │
└────────┴─────────────┴──────────────┴──────┴────────┘
```

## 10.5 3-Tier Traffic Flow

```
3-TIER REQUEST FLOW (User Accessing Web App):
─────────────────────────────────────────────────────────────

REQUEST JOURNEY:
User (Browser)
    │
    │ 1. HTTPS Request to www.example.com
    ▼
Public Load Balancer (20.50.100.1)
    │
    │ 2. Load balances across Web VMs
    ▼
Web VM (10.0.1.4)
    │ • Receives HTTP/HTTPS request
    │ • Serves static content (HTML, CSS, JS)
    │ • Forwards API calls to App Tier
    │
    │ 3. API Call: GET /api/products (Port 8080)
    ▼
Internal Load Balancer (10.0.2.10)
    │
    │ 4. Load balances across App VMs
    ▼
App VM (10.0.2.4)
    │ • Processes business logic
    │ • Validates request
    │ • Queries database
    │
    │ 5. SQL Query (Port 1433)
    ▼
Database Server (10.0.3.4)
    │ • Processes SQL query
    │ • Returns result set
    │
    │ 6. Returns data
    ▼
App VM → Web VM → Load Balancer → User

RESPONSE JOURNEY (Reverse Path):
DB → App VM → Internal LB → Web VM → Public LB → User
```

## 10.6 3-Tier with VNet Peering (Multi-VNet)

```
3-TIER ACROSS MULTIPLE VNets:
─────────────────────────────────────────────────────────────

Use Case: Separate shared services into dedicated VNet

VNet-Prod (10.1.0.0/16)          VNet-Shared (10.2.0.0/16)
┌──────────────────────┐         ┌──────────────────────────┐
│  Web:  10.1.1.0/24  │         │  Monitoring: 10.2.1.0/24 │
│  App:  10.1.2.0/24  │◄───────►│  Identity:  10.2.2.0/24  │
│  DB:   10.1.3.0/24  │ Peering │  DNS:       10.2.3.0/24  │
└──────────────────────┘         └──────────────────────────┘

Peering allows:
• Production VMs to use Shared DNS
• Monitoring tools to access Production metrics
• Identity services to authenticate users
```

---

<a name="implementation"></a>

# Section 11: Implementation Process

## Overview

This section provides step-by-step guidance for implementing:
1. A 3-Tier Application in a single VNet
2. VNet Peering between two VNets

**All steps use the Azure Portal (GUI).**

---

## 11.1 Implementation: 3-Tier Application in Single VNet

### Prerequisites

```
Before starting, ensure you have:
─────────────────────────────────────────────────────────────

✅ Azure Subscription with Contributor access
✅ Resource Group created or permission to create one
✅ Understanding of your IP address plan
✅ Azure Portal access: portal.azure.com

IP ADDRESS PLAN FOR THIS IMPLEMENTATION:
┌──────────────────────────────────────────────────────────┐
│  Resource                │  Value                        │
├──────────────────────────┼───────────────────────────────┤
│  Resource Group          │  RG-ThreeTier-Demo            │
│  Region                  │  East US                      │
│  VNet Name               │  VNet-ThreeTier               │
│  VNet Address Space      │  10.0.0.0/16                  │
│  Web Subnet              │  10.0.1.0/24                  │
│  App Subnet              │  10.0.2.0/24                  │
│  DB Subnet               │  10.0.3.0/24                  │
└──────────────────────────┴───────────────────────────────┘
```

---

### STEP 1: Create Resource Group

```
PURPOSE: Logical container for all related resources

NAVIGATION:
Portal → Home → Resource Groups → + Create

CONFIGURATION:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Field               │  Value                          │
├────────────────────────────────────────────────────────┤
│  Subscription        │  [Your Subscription]            │
│  Resource Group Name │  RG-ThreeTier-Demo              │
│  Region              │  East US                        │
└────────────────────────────────────────────────────────┘

STEPS IN PORTAL:
  1. Click "Home" in left menu
  2. Click "Resource Groups"
  3. Click "+ Create" button
  4. Select your Subscription
  5. Enter Name: RG-ThreeTier-Demo
  6. Select Region: East US
  7. Click "Review + Create"
  8. Click "Create"
```

---

### STEP 2: Create Virtual Network with Subnets

```
PURPOSE: Create the network foundation with three subnets

NAVIGATION:
Portal → Search "Virtual Networks" → + Create

TAB 1 - BASICS:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Field               │  Value                          │
├────────────────────────────────────────────────────────┤
│  Subscription        │  [Your Subscription]            │
│  Resource Group      │  RG-ThreeTier-Demo              │
│  Name                │  VNet-ThreeTier                 │
│  Region              │  East US                        │
└────────────────────────────────────────────────────────┘

TAB 2 - IP ADDRESSES:
─────────────────────────────────────────────────────────────

Address Space Configuration:
┌────────────────────────────────────────────────────────┐
│  IPv4 Address Space  │  10.0.0.0/16                    │
└────────────────────────────────────────────────────────┘

Delete the default subnet and create three new subnets:

SUBNET 1 - Web Tier:
┌────────────────────────────────────────────────────────┐
│  Subnet Name         │  Subnet-Web                     │
│  Subnet Address Range│  10.0.1.0/24                    │
└────────────────────────────────────────────────────────┘

SUBNET 2 - App Tier:
┌────────────────────────────────────────────────────────┐
│  Subnet Name         │  Subnet-App                     │
│  Subnet Address Range│  10.0.2.0/24                    │
└────────────────────────────────────────────────────────┘

SUBNET 3 - Database Tier:
┌────────────────────────────────────────────────────────┐
│  Subnet Name         │  Subnet-DB                      │
│  Subnet Address Range│  10.0.3.0/24                    │
└────────────────────────────────────────────────────────┘

STEPS IN PORTAL:
  1. Search for "Virtual Networks" in Azure Portal
  2. Click "+ Create"
  3. Fill in Basics tab
  4. Navigate to "IP Addresses" tab
  5. Click on "default" subnet → Delete it
  6. Click "+ Add subnet" → Create Subnet-Web (10.0.1.0/24)
  7. Click "+ Add subnet" → Create Subnet-App (10.0.2.0/24)
  8. Click "+ Add subnet" → Create Subnet-DB (10.0.3.0/24)
  9. Click "Review + Create" → "Create"
```

---

### STEP 3: Create Network Security Groups

```
PURPOSE: Define firewall rules for each tier

NAVIGATION:
Portal → Search "Network Security Groups" → + Create

CREATE NSG 1 - WEB TIER:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Field               │  Value                          │
├────────────────────────────────────────────────────────┤
│  Resource Group      │  RG-ThreeTier-Demo              │
│  Name                │  NSG-Web                        │
│  Region              │  East US                        │
└────────────────────────────────────────────────────────┘

After creating NSG-Web, add inbound rules:

INBOUND RULE 1 - Allow HTTP:
┌────────────────────────────────────────────────────────┐
│  Source              │  Service Tag                    │
│  Source Service Tag  │  Internet                       │
│  Source Port Ranges  │  *                              │
│  Destination         │  Any                            │
│  Service             │  HTTP                           │
│  Destination Port    │  80                             │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  100                            │
│  Name                │  Allow-HTTP-Inbound             │
└────────────────────────────────────────────────────────┘

INBOUND RULE 2 - Allow HTTPS:
┌────────────────────────────────────────────────────────┐
│  Source              │  Service Tag                    │
│  Source Service Tag  │  Internet                       │
│  Source Port Ranges  │  *                              │
│  Destination         │  Any                            │
│  Service             │  HTTPS                          │
│  Destination Port    │  443                            │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  110                            │
│  Name                │  Allow-HTTPS-Inbound            │
└────────────────────────────────────────────────────────┘

OUTBOUND RULE 1 - Allow to App Tier:
┌────────────────────────────────────────────────────────┐
│  Source              │  IP Addresses                   │
│  Source IP Ranges    │  10.0.1.0/24                    │
│  Source Port Ranges  │  *                              │
│  Destination         │  IP Addresses                   │
│  Destination IP      │  10.0.2.0/24                    │
│  Destination Port    │  8080                           │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  100                            │
│  Name                │  Allow-To-AppTier               │
└────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════

CREATE NSG 2 - APP TIER:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  NSG-App                        │
│  Region              │  East US                        │
│  Resource Group      │  RG-ThreeTier-Demo              │
└────────────────────────────────────────────────────────┘

INBOUND RULE - Allow from Web Tier only:
┌────────────────────────────────────────────────────────┐
│  Source              │  IP Addresses                   │
│  Source IP Ranges    │  10.0.1.0/24                    │
│  Source Port Ranges  │  *                              │
│  Destination         │  Any                            │
│  Destination Port    │  8080                           │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  100                            │
│  Name                │  Allow-From-WebTier             │
└────────────────────────────────────────────────────────┘

OUTBOUND RULE - Allow to DB Tier:
┌────────────────────────────────────────────────────────┐
│  Source              │  IP Addresses                   │
│  Source IP Ranges    │  10.0.2.0/24                    │
│  Source Port Ranges  │  *                              │
│  Destination         │  IP Addresses                   │
│  Destination IP      │  10.0.3.0/24                    │
│  Destination Port    │  1433                           │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  100                            │
│  Name                │  Allow-To-DBTier                │
└────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════

CREATE NSG 3 - DB TIER:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  NSG-DB                         │
│  Region              │  East US                        │
│  Resource Group      │  RG-ThreeTier-Demo              │
└────────────────────────────────────────────────────────┘

INBOUND RULE - Allow from App Tier only:
┌────────────────────────────────────────────────────────┐
│  Source              │  IP Addresses                   │
│  Source IP Ranges    │  10.0.2.0/24                    │
│  Source Port Ranges  │  *                              │
│  Destination         │  Any                            │
│  Destination Port    │  1433                           │
│  Protocol            │  TCP                            │
│  Action              │  Allow                          │
│  Priority            │  100                            │
│  Name                │  Allow-From-AppTier-SQL         │
└────────────────────────────────────────────────────────┘
```

---

### STEP 4: Associate NSGs with Subnets

```
PURPOSE: Apply firewall rules to each subnet

NAVIGATION:
Portal → NSG-Web → Settings → Subnets → + Associate

ASSOCIATE NSG-Web WITH Subnet-Web:
─────────────────────────────────────────────────────────────

  1. Go to "Network Security Groups"
  2. Click on "NSG-Web"
  3. In left menu, click "Subnets" under Settings
  4. Click "+ Associate"
  5. Select:
     Virtual Network: VNet-ThreeTier
     Subnet:          Subnet-Web
  6. Click "OK"

ASSOCIATE NSG-App WITH Subnet-App:
─────────────────────────────────────────────────────────────

  1. Go to "Network Security Groups"
  2. Click on "NSG-App"
  3. Click "Subnets" → "+ Associate"
  4. Select:
     Virtual Network: VNet-ThreeTier
     Subnet:          Subnet-App
  5. Click "OK"

ASSOCIATE NSG-DB WITH Subnet-DB:
─────────────────────────────────────────────────────────────

  1. Go to "Network Security Groups"
  2. Click on "NSG-DB"
  3. Click "Subnets" → "+ Associate"
  4. Select:
     Virtual Network: VNet-ThreeTier
     Subnet:          Subnet-DB
  5. Click "OK"

VERIFICATION:
─────────────────────────────────────────────────────────────
Go to VNet-ThreeTier → Subnets
Each subnet should show its associated NSG:
✅ Subnet-Web → NSG-Web
✅ Subnet-App → NSG-App
✅ Subnet-DB  → NSG-DB
```

---

### STEP 5: Deploy Web Tier Virtual Machines

```
PURPOSE: Create web servers in the Web Tier subnet

NAVIGATION:
Portal → Virtual Machines → + Create → Azure Virtual Machine

VM CONFIGURATION - Web VM 1:
─────────────────────────────────────────────────────────────

BASICS TAB:
┌────────────────────────────────────────────────────────┐
│  Field                   │  Value                      │
├────────────────────────────────────────────────────────┤
│  Subscription            │  [Your Subscription]        │
│  Resource Group          │  RG-ThreeTier-Demo          │
│  VM Name                 │  VM-Web-01                  │
│  Region                  │  East US                    │
│  Availability Options    │  Availability Zone          │
│  Zone                    │  Zone 1                     │
│  Image                   │  Windows Server 2022        │
│  Size                    │  Standard_B2s               │
│  Username                │  azureadmin                 │
│  Password                │  [Strong Password]          │
│  Public Inbound Ports    │  None (NSG will handle it)  │
└────────────────────────────────────────────────────────┘

NETWORKING TAB:
┌────────────────────────────────────────────────────────┐
│  Field                   │  Value                      │
├────────────────────────────────────────────────────────┤
│  Virtual Network         │  VNet-ThreeTier             │
│  Subnet                  │  Subnet-Web (10.0.1.0/24)   │
│  Public IP               │  None                       │
│  NIC NSG                 │  None (Using Subnet NSG)    │
│  Load Balancing          │  None (configure separately)│
└────────────────────────────────────────────────────────┘

  1. Click "Review + Create"
  2. Validate configuration
  3. Click "Create"
  4. Wait for deployment (~3-5 minutes)

REPEAT FOR Web VM 2:
  • Name: VM-Web-02
  • Zone: Zone 2 (for high availability)
  • Same subnet: Subnet-Web
```

---

### STEP 6: Deploy Application Tier Virtual Machines

```
PURPOSE: Create application servers in App Tier subnet

VM CONFIGURATION - App VM 1:
─────────────────────────────────────────────────────────────

BASICS TAB:
┌────────────────────────────────────────────────────────┐
│  Field                   │  Value                      │
├────────────────────────────────────────────────────────┤
│  Resource Group          │  RG-ThreeTier-Demo          │
│  VM Name                 │  VM-App-01                  │
│  Region                  │  East US                    │
│  Availability Zone       │  Zone 1                     │
│  Image                   │  Windows Server 2022        │
│  Size                    │  Standard_B2s               │
│  Username                │  azureadmin                 │
│  Password                │  [Strong Password]          │
│  Public Inbound Ports    │  None                       │
└────────────────────────────────────────────────────────┘

NETWORKING TAB:
┌────────────────────────────────────────────────────────┐
│  Virtual Network         │  VNet-ThreeTier             │
│  Subnet                  │  Subnet-App (10.0.2.0/24)   │
│  Public IP               │  None                       │
│  NIC NSG                 │  None                       │
└────────────────────────────────────────────────────────┘

REPEAT FOR App VM 2:
  • Name: VM-App-02
  • Zone: Zone 2
  • Same subnet: Subnet-App
```

---

### STEP 7: Deploy Database Tier

```
PURPOSE: Create database server in DB Tier subnet

OPTION A: Azure SQL Database (Recommended for PaaS)
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → Search "SQL Databases" → + Create

CONFIGURATION:
┌────────────────────────────────────────────────────────┐
│  Field                   │  Value                      │
├────────────────────────────────────────────────────────┤
│  Resource Group          │  RG-ThreeTier-Demo          │
│  Database Name           │  DB-ThreeTier               │
│  Server                  │  Create New                 │
│  Server Name             │  sql-threetier-demo         │
│  Authentication          │  SQL Authentication         │
│  Admin Login             │  sqladmin                   │
│  Password                │  [Strong Password]          │
│  Compute + Storage       │  General Purpose - Standard │
└────────────────────────────────────────────────────────┘

NETWORKING TAB:
┌────────────────────────────────────────────────────────┐
│  Connectivity Method     │  Private Endpoint           │
│  Virtual Network         │  VNet-ThreeTier             │
│  Subnet                  │  Subnet-DB                  │
└────────────────────────────────────────────────────────┘

OPTION B: SQL Server on Virtual Machine (IaaS)
─────────────────────────────────────────────────────────────

VM CONFIGURATION - DB Server:
┌────────────────────────────────────────────────────────┐
│  VM Name                 │  VM-DB-01                   │
│  Image                   │  SQL Server 2022 on Win2022 │
│  Size                    │  Standard_D4s_v3            │
│  Subnet                  │  Subnet-DB (10.0.3.0/24)    │
│  Public IP               │  None                       │
└────────────────────────────────────────────────────────┘
```

---

### STEP 8: Create Load Balancer (Public - Web Tier)

```
PURPOSE: Distribute internet traffic across Web VMs

NAVIGATION:
Portal → Search "Load Balancers" → + Create

TAB 1 - BASICS:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Field                   │  Value                      │
├────────────────────────────────────────────────────────┤
│  Resource Group          │  RG-ThreeTier-Demo          │
│  Name                    │  LB-Web-Public              │
│  Region                  │  East US                    │
│  SKU                     │  Standard                   │
│  Type                    │  Public                     │
│  Tier                    │  Regional                   │
└────────────────────────────────────────────────────────┘

TAB 2 - FRONTEND IP CONFIGURATION:
─────────────────────────────────────────────────────────────

  Click "+ Add a frontend IP configuration"
┌────────────────────────────────────────────────────────┐
│  Name                    │  Frontend-Web               │
│  IP Version              │  IPv4                       │
│  IP Type                 │  IP Address                 │
│  Public IP Address       │  Create New                 │
│  Public IP Name          │  PIP-Web-LB                 │
│  Availability Zone       │  Zone-Redundant             │
└────────────────────────────────────────────────────────┘

TAB 3 - BACKEND POOLS:
─────────────────────────────────────────────────────────────

  Click "+ Add a backend pool"
┌────────────────────────────────────────────────────────┐
│  Name                    │  BackendPool-Web            │
│  Virtual Network         │  VNet-ThreeTier             │
│  Backend Pool Config     │  NIC                        │
└────────────────────────────────────────────────────────┘

  Add VMs to backend pool:
  • VM-Web-01 → NIC → IP Configuration
  • VM-Web-02 → NIC → IP Configuration

TAB 4 - INBOUND RULES:
─────────────────────────────────────────────────────────────

  Click "+ Add a load balancing rule"
┌────────────────────────────────────────────────────────┐
│  Name                    │  Rule-HTTP                  │
│  Frontend IP             │  Frontend-Web               │
│  Backend Pool            │  BackendPool-Web            │
│  Protocol                │  TCP                        │
│  Port                    │  80                         │
│  Backend Port            │  80                         │
│  Health Probe            │  Create New                 │
│  Health Probe Name       │  Probe-HTTP                 │
│  Health Probe Protocol   │  HTTP                       │
│  Health Probe Port       │  80                         │
│  Health Probe Path       │  /                          │
│  Session Persistence     │  None                       │
└────────────────────────────────────────────────────────┘

  Add second rule for HTTPS:
┌────────────────────────────────────────────────────────┐
│  Name                    │  Rule-HTTPS                 │
│  Port                    │  443                        │
│  Backend Port            │  443                        │
└────────────────────────────────────────────────────────┘

  Click "Review + Create" → "Create"
```

---

### STEP 9: Create Internal Load Balancer (App Tier)

```
PURPOSE: Distribute traffic from Web Tier to App VMs

NAVIGATION:
Portal → Load Balancers → + Create

BASICS:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                    │  LB-App-Internal            │
│  SKU                     │  Standard                   │
│  Type                    │  Internal                   │
└────────────────────────────────────────────────────────┘

FRONTEND IP:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                    │  Frontend-App               │
│  Virtual Network         │  VNet-ThreeTier             │
│  Subnet                  │  Subnet-App                 │
│  IP Assignment           │  Static                     │
│  IP Address              │  10.0.2.10                  │
└────────────────────────────────────────────────────────┘

BACKEND POOL:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                    │  BackendPool-App            │
│  Add VMs                 │  VM-App-01, VM-App-02       │
└────────────────────────────────────────────────────────┘

LOAD BALANCING RULE:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                    │  Rule-AppAPI                │
│  Frontend IP             │  Frontend-App (10.0.2.10)   │
│  Backend Pool            │  BackendPool-App            │
│  Protocol                │  TCP                        │
│  Port                    │  8080                       │
│  Backend Port            │  8080                       │
└────────────────────────────────────────────────────────┘
```

---

### STEP 10: Verify 3-Tier Connectivity

```
VERIFICATION STEPS:
─────────────────────────────────────────────────────────────

STEP 10.1: Verify NSG Rules Applied
  1. Go to VNet-ThreeTier → Subnets
  2. Click each subnet → Verify NSG is associated
  3. Check: Subnet-Web shows NSG-Web ✅
  4. Check: Subnet-App shows NSG-App ✅
  5. Check: Subnet-DB shows NSG-DB ✅

STEP 10.2: Test Connectivity Using Azure Network Watcher
  1. Search "Network Watcher" in portal
  2. Click "Connection Monitor" → "+ Create"
  3. Test 1: Web VM → App VM on Port 8080 (should succeed)
  4. Test 2: Web VM → DB VM on Port 1433 (should fail - NSG blocks)
  5. Test 3: App VM → DB VM on Port 1433 (should succeed)

STEP 10.3: Verify IP Flow
  1. Network Watcher → IP Flow Verify
  2. Test: Internet → Web VM Port 443 → Should: ALLOW ✅
  3. Test: Internet → App VM Port 8080 → Should: DENY ❌
  4. Test: Internet → DB VM Port 1433 → Should: DENY ❌

EXPECTED RESULTS:
─────────────────────────────────────────────────────────────
┌──────────────────────────────────────────────┬──────────┐
│  Test                                        │ Result   │
├──────────────────────────────────────────────┼──────────┤
│  Internet → Web VM (Port 443)                │ ✅ Allow │
│  Internet → App VM (Port 8080)               │ ❌ Deny  │
│  Internet → DB VM (Port 1433)                │ ❌ Deny  │
│  Web VM → App VM (Port 8080)                 │ ✅ Allow │
│  Web VM → DB VM (Port 1433)                  │ ❌ Deny  │
│  App VM → DB VM (Port 1433)                  │ ✅ Allow │
│  DB VM → Internet (Any)                      │ ❌ Deny  │
└──────────────────────────────────────────────┴──────────┘
```

---

## 11.2 Implementation: VNet Peering

### Scenario Overview

```
VNet PEERING SCENARIO:
─────────────────────────────────────────────────────────────

We will create two VNets in the same region and peer them:

VNet-A (Production)               VNet-B (Shared Services)
10.1.0.0/16                       10.2.0.0/16
┌───────────────────┐             ┌───────────────────────┐
│ Web: 10.1.1.0/24 │             │ Monitor: 10.2.1.0/24  │
│ App: 10.1.2.0/24 │◄──Peering──►│ DNS:     10.2.2.0/24  │
│ DB:  10.1.3.0/24 │             └───────────────────────┘
└───────────────────┘

GOAL: VNet-A resources can communicate with VNet-B services
```

---

### STEP 1: Create VNet-A (Production)

```
NAVIGATION:
Portal → Virtual Networks → + Create

CONFIGURATION VNet-A:
─────────────────────────────────────────────────────────────

BASICS TAB:
┌────────────────────────────────────────────────────────┐
│  Resource Group      │  RG-Peering-Demo                │
│  Name                │  VNet-A-Production              │
│  Region              │  East US                        │
└────────────────────────────────────────────────────────┘

IP ADDRESSES TAB:
┌────────────────────────────────────────────────────────┐
│  IPv4 Address Space  │  10.1.0.0/16                    │
└────────────────────────────────────────────────────────┘

SUBNETS:
┌────────────────────────────────────────────────────────┐
│  Subnet 1 Name       │  Subnet-Web-A                   │
│  Subnet 1 Range      │  10.1.1.0/24                    │
├────────────────────────────────────────────────────────┤
│  Subnet 2 Name       │  Subnet-App-A                   │
│  Subnet 2 Range      │  10.1.2.0/24                    │
└────────────────────────────────────────────────────────┘

  Click "Review + Create" → "Create"
```

---

### STEP 2: Create VNet-B (Shared Services)

```
NAVIGATION:
Portal → Virtual Networks → + Create

CONFIGURATION VNet-B:
─────────────────────────────────────────────────────────────

BASICS TAB:
┌────────────────────────────────────────────────────────┐
│  Resource Group      │  RG-Peering-Demo                │
│  Name                │  VNet-B-Shared                  │
│  Region              │  East US                        │
└────────────────────────────────────────────────────────┘

IP ADDRESSES TAB:
┌────────────────────────────────────────────────────────┐
│  IPv4 Address Space  │  10.2.0.0/16                    │
└────────────────────────────────────────────────────────┘

SUBNETS:
┌────────────────────────────────────────────────────────┐
│  Subnet 1 Name       │  Subnet-Monitor                 │
│  Subnet 1 Range      │  10.2.1.0/24                    │
├────────────────────────────────────────────────────────┤
│  Subnet 2 Name       │  Subnet-DNS                     │
│  Subnet 2 Range      │  10.2.2.0/24                    │
└────────────────────────────────────────────────────────┘

  Click "Review + Create" → "Create"
```

---

### STEP 3: Deploy Test Virtual Machines

```
PURPOSE: Create VMs to test peering connectivity

VM IN VNet-A (Production):
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  VM-Prod-01                     │
│  Resource Group      │  RG-Peering-Demo                │
│  Region              │  East US                        │
│  Image               │  Ubuntu Server 22.04 LTS        │
│  Size                │  Standard_B1s                   │
│  VNet                │  VNet-A-Production              │
│  Subnet              │  Subnet-Web-A (10.1.1.0/24)     │
│  Public IP           │  None                           │
└────────────────────────────────────────────────────────┘

VM IN VNet-B (Shared Services):
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  VM-Shared-01                   │
│  Resource Group      │  RG-Peering-Demo                │
│  Region              │  East US                        │
│  Image               │  Ubuntu Server 22.04 LTS        │
│  Size                │  Standard_B1s                   │
│  VNet                │  VNet-B-Shared                  │
│  Subnet              │  Subnet-Monitor (10.2.1.0/24)   │
│  Public IP           │  None                           │
└────────────────────────────────────────────────────────┘

NOTE THE PRIVATE IP ADDRESSES:
  VM-Prod-01:   Should be 10.1.1.4
  VM-Shared-01: Should be 10.2.1.4
```

---

### STEP 4: Configure VNet Peering (VNet-A to VNet-B)

```
IMPORTANT: Peering requires TWO connections:
  Connection 1: VNet-A → VNet-B
  Connection 2: VNet-B → VNet-A

NAVIGATION:
Portal → Virtual Networks → VNet-A-Production → Peerings

CREATE PEERING FROM VNet-A TO VNet-B:
─────────────────────────────────────────────────────────────

  1. Go to VNet-A-Production
  2. In left menu, scroll to "Settings" → Click "Peerings"
  3. Click "+ Add"

PEERING CONFIGURATION FORM:

This Virtual Network (VNet-A) Section:
┌────────────────────────────────────────────────────────┐
│  Field                         │  Value                │
├────────────────────────────────┼───────────────────────┤
│  Peering Link Name             │  VNetA-to-VNetB       │
│  Traffic to remote VNet        │  Allow                │
│  Traffic from remote VNet      │  Allow                │
│  Virtual Network Gateway       │  None                 │
└────────────────────────────────┴───────────────────────┘

Remote Virtual Network (VNet-B) Section:
┌────────────────────────────────────────────────────────┐
│  Field                         │  Value                │
├────────────────────────────────┼───────────────────────┤
│  Peering Link Name             │  VNetB-to-VNetA       │
│  Virtual Network Deployment    │  Resource Manager     │
│  Subscription                  │  [Your Subscription]  │
│  Virtual Network               │  VNet-B-Shared        │
│  Traffic to remote VNet        │  Allow                │
│  Traffic from remote VNet      │  Allow                │
│  Virtual Network Gateway       │  None                 │
└────────────────────────────────┴───────────────────────┘

  4. Click "Add"
  5. Wait for peering status to show "Connected"

VERIFY PEERING STATUS:
─────────────────────────────────────────────────────────────

After creation, you should see in VNet-A Peerings:
┌──────────────────┬──────────────┬──────────────────────┐
│  Peering Name    │  Status      │  Remote VNet         │
├──────────────────┼──────────────┼──────────────────────┤
│  VNetA-to-VNetB  │  Connected ✅│  VNet-B-Shared       │
└──────────────────┴──────────────┴──────────────────────┘

And in VNet-B Peerings:
┌──────────────────┬──────────────┬──────────────────────┐
│  Peering Name    │  Status      │  Remote VNet         │
├──────────────────┼──────────────┼──────────────────────┤
│  VNetB-to-VNetA  │  Connected ✅│  VNet-A-Production   │
└──────────────────┴──────────────┴──────────────────────┘

STATUS MEANINGS:
  • Initiated:   One side created, waiting for other side
  • Connected:   Both sides created and working ✅
  • Disconnected: Problem - check configuration ❌
```

---

### STEP 5: Verify Peering - Routing Table Check

```
VERIFY ROUTING TABLES UPDATED:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → VNet-A-Production → Subnets → Subnet-Web-A
→ View Route Table

OR

Portal → Network Watcher → Effective Routes
→ Select VM-Prod-01 NIC

Expected Effective Routes on VM-Prod-01 (VNet-A):
┌──────────────┬──────────────────────────────────────────┐
│ Destination  │ Next Hop                                 │
├──────────────┼──────────────────────────────────────────┤
│ 10.1.0.0/16  │ Virtual Network (local)                  │
│ 10.2.0.0/16  │ VNet Peering ← SHOULD APPEAR AFTER PEER │
│ 0.0.0.0/0    │ Internet                                 │
└──────────────┴──────────────────────────────────────────┘

If 10.2.0.0/16 with "VNet Peering" next hop appears = ✅ SUCCESS
```

---

### STEP 6: Test Peering Connectivity

```
TEST METHOD 1: Azure Network Watcher - Connection Monitor
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → Network Watcher → Connection Monitor → + Create

CONFIGURATION:
┌────────────────────────────────────────────────────────┐
│  Name               │  Test-VNetA-to-VNetB             │
│  Region             │  East US                         │
└────────────────────────────────────────────────────────┘

SOURCE:
┌────────────────────────────────────────────────────────┐
│  Source Type        │  Virtual Machine                 │
│  Virtual Machine    │  VM-Prod-01 (in VNet-A)          │
└────────────────────────────────────────────────────────┘

DESTINATION + TEST:
┌────────────────────────────────────────────────────────┐
│  Destination Type   │  Virtual Machine                 │
│  Destination VM     │  VM-Shared-01 (in VNet-B)        │
│  Port               │  22 (SSH) or ICMP               │
└────────────────────────────────────────────────────────┘

  Click "Create" → Wait for test results
  Expected: Connection successful ✅

TEST METHOD 2: Azure Network Watcher - IP Flow Verify
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → Network Watcher → IP Flow Verify

┌────────────────────────────────────────────────────────┐
│  Virtual Machine    │  VM-Prod-01                      │
│  Direction          │  Outbound                        │
│  Protocol           │  TCP                             │
│  Local Port         │  60000                           │
│  Remote IP          │  10.2.1.4 (VM-Shared-01 IP)      │
│  Remote Port        │  22                              │
└────────────────────────────────────────────────────────┘

  Expected: "Access allowed" ✅

EXPECTED TEST RESULTS:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────┬────────┐
│  Test                                          │ Result │
├────────────────────────────────────────────────┼────────┤
│  VM-Prod-01 → VM-Shared-01 (Peered)            │ ✅     │
│  VM-Shared-01 → VM-Prod-01 (Peered)            │ ✅     │
│  VM-Prod-01 → 10.3.0.4 (Non-Peered VNet)       │ ❌     │
└────────────────────────────────────────────────┴────────┘
```

---

### STEP 7: Configure NSGs for Peered Traffic

```
PURPOSE: Allow specific traffic between peered VNets

SCENARIO: Allow monitoring VM (VNet-B) to ping/monitor VNet-A VMs

ADD NSG RULE TO VNet-A SUBNET NSG:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → NSG (associated with Subnet-Web-A) → 
Inbound Security Rules → + Add

RULE CONFIGURATION:
┌────────────────────────────────────────────────────────┐
│  Source              │  IP Addresses                   │
│  Source IP Ranges    │  10.2.0.0/16                    │
│  Source Port Ranges  │  *                              │
│  Destination         │  IP Addresses                   │
│  Destination IP      │  10.1.0.0/16                    │
│  Destination Port    │  *                              │
│  Protocol            │  ICMP                           │
│  Action              │  Allow                          │
│  Priority            │  200                            │
│  Name                │  Allow-From-SharedVNet-Ping     │
└────────────────────────────────────────────────────────┘

NOTE: Without this rule, even with peering, ICMP would
      be blocked by the Deny-All-Inbound default rule

IMPORTANT CONCEPT:
─────────────────────────────────────────────────────────────

VNet Peering enables ROUTING between VNets
NSGs control what TRAFFIC is actually allowed

Both must be configured correctly!

Peering = Opens the "road" between VNets
NSG = The "traffic rules" on that road
```

---

## 11.3 Implementation: Hub-Spoke with Transitive Routing

### Scenario Overview

```
HUB-SPOKE SCENARIO:
─────────────────────────────────────────────────────────────

Hub VNet (10.0.0.0/16)
├── Hub Subnet: 10.0.1.0/24 (Contains Azure Firewall/NVA)
│
├── Peering to Spoke-A VNet (10.1.0.0/16)
│   └── Spoke-A Subnet: 10.1.1.0/24
│
└── Peering to Spoke-B VNet (10.2.0.0/16)
    └── Spoke-B Subnet: 10.2.1.0/24

GOAL: Spoke-A VMs can communicate with Spoke-B VMs
      through the Hub (transitive routing)
```

---

### STEP 1: Create Hub VNet

```
NAVIGATION:
Portal → Virtual Networks → + Create

CONFIGURATION:
┌────────────────────────────────────────────────────────┐
│  Name                │  VNet-Hub                       │
│  Resource Group      │  RG-HubSpoke-Demo               │
│  Region              │  East US                        │
│  Address Space       │  10.0.0.0/16                    │
└────────────────────────────────────────────────────────┘

SUBNETS:
┌────────────────────────────────────────────────────────┐
│  Hub Subnet          │  10.0.1.0/24                    │
│  AzureFirewallSubnet │  10.0.2.0/26 (Required name!)   │
└────────────────────────────────────────────────────────┘

IMPORTANT: The Azure Firewall subnet MUST be named
           "AzureFirewallSubnet" exactly
```

---

### STEP 2: Create Spoke VNets

```
CREATE VNet-Spoke-A:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  VNet-Spoke-A                   │
│  Address Space       │  10.1.0.0/16                    │
│  Subnet Name         │  Subnet-Spoke-A                 │
│  Subnet Range        │  10.1.1.0/24                    │
└────────────────────────────────────────────────────────┘

CREATE VNet-Spoke-B:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  VNet-Spoke-B                   │
│  Address Space       │  10.2.0.0/16                    │
│  Subnet Name         │  Subnet-Spoke-B                 │
│  Subnet Range        │  10.2.1.0/24                    │
└────────────────────────────────────────────────────────┘
```

---

### STEP 3: Deploy Azure Firewall in Hub

```
PURPOSE: Azure Firewall will act as the NVA for routing 
         between spokes

NAVIGATION:
Portal → Search "Firewalls" → + Create

CONFIGURATION:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Field               │  Value                          │
├────────────────────────────────────────────────────────┤
│  Resource Group      │  RG-HubSpoke-Demo               │
│  Name                │  AzFirewall-Hub                 │
│  Region              │  East US                        │
│  Availability Zone   │  None                           │
│  Firewall SKU        │  Standard                       │
│  Firewall Policy     │  Create New: FWPolicy-Hub       │
│  Virtual Network     │  VNet-Hub                       │
│  Public IP Address   │  Create New: PIP-Firewall       │
└────────────────────────────────────────────────────────┘

  Click "Review + Create" → "Create"
  
  NOTE: Azure Firewall takes ~5-10 minutes to deploy

RECORD THE FIREWALL PRIVATE IP:
  After deployment → Click AzFirewall-Hub → 
  Overview → Note "Private IP Address"
  Example: 10.0.2.4 (This will be your NVA IP for routing)
```

---

### STEP 4: Create Hub-Spoke Peerings

```
PEERING 1: Hub ↔ Spoke-A
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → VNet-Hub → Peerings → + Add

This VNet (Hub) Configuration:
┌────────────────────────────────────────────────────────┐
│  Peering Link Name   │  Hub-to-SpokeA                  │
│  Traffic to remote   │  Allow                          │
│  Traffic from remote │  Allow                          │
│  Gateway Transit     │  Allow gateway transit: OFF     │
└────────────────────────────────────────────────────────┘

Remote VNet (Spoke-A) Configuration:
┌────────────────────────────────────────────────────────┐
│  Peering Link Name   │  SpokeA-to-Hub                  │
│  Virtual Network     │  VNet-Spoke-A                   │
│  Traffic to remote   │  Allow                          │
│  Traffic from remote │  Allow                          │
│  Use Remote Gateway  │  OFF (No gateway deployed)      │
└────────────────────────────────────────────────────────┘

  Click "Add"

═══════════════════════════════════════════════════════════════

PEERING 2: Hub ↔ Spoke-B
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → VNet-Hub → Peerings → + Add

This VNet (Hub) Configuration:
┌────────────────────────────────────────────────────────┐
│  Peering Link Name   │  Hub-to-SpokeB                  │
│  Traffic to remote   │  Allow                          │
│  Traffic from remote │  Allow                          │
└────────────────────────────────────────────────────────┘

Remote VNet (Spoke-B) Configuration:
┌────────────────────────────────────────────────────────┐
│  Peering Link Name   │  SpokeB-to-Hub                  │
│  Virtual Network     │  VNet-Spoke-B                   │
│  Traffic to remote   │  Allow                          │
│  Traffic from remote │  Allow                          │
└────────────────────────────────────────────────────────┘

  Click "Add"
```

---

### STEP 5: Create User-Defined Routes (UDR) for Transitive Routing

```
PURPOSE: Force spoke-to-spoke traffic through Hub Firewall

CREATE ROUTE TABLE FOR SPOKE-A:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → Search "Route Tables" → + Create

BASICS:
┌────────────────────────────────────────────────────────┐
│  Name                │  RT-SpokeA                      │
│  Resource Group      │  RG-HubSpoke-Demo               │
│  Region              │  East US                        │
│  Propagate gateway   │  Yes                            │
└────────────────────────────────────────────────────────┘

  Click "Create"

ADD ROUTES TO RT-SpokeA:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → RT-SpokeA → Settings → Routes → + Add

ROUTE 1 - Send Spoke-B traffic through Firewall:
┌────────────────────────────────────────────────────────┐
│  Route Name          │  To-SpokeB-via-Firewall         │
│  Destination Type    │  IP Addresses                   │
│  Destination CIDR    │  10.2.0.0/16                    │
│  Next Hop Type       │  Virtual Appliance              │
│  Next Hop IP Address │  10.0.2.4 (Firewall Private IP) │
└────────────────────────────────────────────────────────┘

  Click "Add"

ASSOCIATE RT-SpokeA WITH Subnet-Spoke-A:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → RT-SpokeA → Settings → Subnets → + Associate

┌────────────────────────────────────────────────────────┐
│  Virtual Network     │  VNet-Spoke-A                   │
│  Subnet              │  Subnet-Spoke-A                 │
└────────────────────────────────────────────────────────┘

  Click "OK"

═══════════════════════════════════════════════════════════════

CREATE ROUTE TABLE FOR SPOKE-B:
─────────────────────────────────────────────────────────────

NAVIGATION:
Portal → Route Tables → + Create

BASICS:
┌────────────────────────────────────────────────────────┐
│  Name                │  RT-SpokeB                      │
└────────────────────────────────────────────────────────┘

ADD ROUTE:
┌────────────────────────────────────────────────────────┐
│  Route Name          │  To-SpokeA-via-Firewall         │
│  Destination CIDR    │  10.1.0.0/16                    │
│  Next Hop Type       │  Virtual Appliance              │
│  Next Hop IP         │  10.0.2.4 (Firewall Private IP) │
└────────────────────────────────────────────────────────┘

ASSOCIATE WITH Subnet-Spoke-B:
┌────────────────────────────────────────────────────────┐
│  Virtual Network     │  VNet-Spoke-B                   │
│  Subnet              │  Subnet-Spoke-B                 │
└────────────────────────────────────────────────────────┘
```

---

### STEP 6: Configure Azure Firewall Policy Rules

```
PURPOSE: Allow traffic between spokes through firewall

NAVIGATION:
Portal → Firewall Policies → FWPolicy-Hub → 
Network Rule Collections → + Add

NETWORK RULE COLLECTION:
─────────────────────────────────────────────────────────────
┌────────────────────────────────────────────────────────┐
│  Name                │  Allow-Spoke-to-Spoke           │
│  Rule Collection Type│  Network                        │
│  Priority            │  100                            │
│  Action              │  Allow                          │
└────────────────────────────────────────────────────────┘

ADD RULE:
┌────────────────────────────────────────────────────────┐
│  Name                │  Allow-SpokeA-to-SpokeB         │
│  Source Type         │  IP Address                     │
│  Source              │  10.1.0.0/16                    │
│  Protocol            │  TCP,UDP,ICMP                   │
│  Destination Ports   │  *                              │
│  Destination Type    │  IP Address                     │
│  Destination         │  10.2.0.0/16                    │
└────────────────────────────────────────────────────────┘

ADD RULE 2:
┌────────────────────────────────────────────────────────┐
│  Name                │  Allow-SpokeB-to-SpokeA         │
│  Source              │  10.2.0.0/16                    │
│  Protocol            │  TCP,UDP,ICMP                   │
│  Destination Ports   │  *                              │
│  Destination         │  10.1.0.0/16                    │
└────────────────────────────────────────────────────────┘

  Click "Add"
```

---

### STEP 7: Verify Hub-Spoke Transitive Routing

```
VERIFICATION STEPS:
─────────────────────────────────────────────────────────────

STEP 7.1: Check Effective Routes on Spoke-A VM
  Portal → VM in Spoke-A → Networking → 
  Network Interface → Effective Routes
  
  Should see:
  ┌──────────────┬──────────────────────────────────────┐
  │ 10.1.0.0/16  │ Virtual Network                      │
  │ 10.0.0.0/16  │ VNet Peering (Hub)                   │
  │ 10.2.0.0/16  │ User Defined → 10.0.2.4 (Firewall)  │
  └──────────────┴──────────────────────────────────────┘

STEP 7.2: Test Spoke-A to Spoke-B via Network Watcher
  Portal → Network Watcher → Connection Monitor
  
  Source: VM-SpokeA (10.1.1.4)
  Destination: VM-SpokeB (10.2.1.4)
  Port: ICMP or 22
  
  Expected: Connected ✅ (Traffic routed through Firewall)

STEP 7.3: Verify in Azure Firewall Logs
  Portal → AzFirewall-Hub → Monitoring → Logs
  
  Check Network Rules Log for entries showing:
  Source: 10.1.x.x → Destination: 10.2.x.x → Allow

TRAFFIC FLOW CONFIRMATION:
─────────────────────────────────────────────────────────────

VM-SpokeA (10.1.1.4)
    │
    │ UDR: 10.2.0.0/16 → 10.0.2.4
    ▼
Azure Firewall (10.0.2.4) in Hub
    │
    │ Firewall Rule: Allow Spoke-to-Spoke
    ▼
VM-SpokeB (10.2.1.4)
    ✅ Connection Successful!
```

---

## 11.4 Troubleshooting Guide

```
COMMON ISSUES AND SOLUTIONS:
─────────────────────────────────────────────────────────────

ISSUE 1: VNet Peering Status "Disconnected"
─────────────────────────────────────────────────────────────
Symptoms: Peering shows "Disconnected" or "Initiated"
Cause:    Only one side of the peering was created

Solution:
  1. Go to both VNets → Peerings
  2. Verify BOTH sides show "Connected"
  3. If one shows "Initiated" → The other side is missing
  4. Delete and recreate the peering from the other VNet

ISSUE 2: Cannot Reach Peered VNet VMs
─────────────────────────────────────────────────────────────
Symptoms: Ping/connection fails despite peering being "Connected"
Cause:    NSG blocking traffic from remote VNet

Solution:
  1. Network Watcher → IP Flow Verify
  2. Identify which NSG is blocking
  3. Add inbound rule allowing source from peered VNet CIDR
  4. Example: Allow from 10.2.0.0/16 on required port

ISSUE 3: Overlapping Address Spaces
─────────────────────────────────────────────────────────────
Symptoms: Peering creation fails with CIDR conflict error
Cause:    Both VNets have same or overlapping address ranges

Solution:
  1. Check both VNet address spaces
  2. One or both must be changed to non-overlapping ranges
  3. May require deleting VMs and recreating VNets
  4. Plan address spaces BEFORE deployment!

ISSUE 4: Hub-Spoke Traffic Not Flowing
─────────────────────────────────────────────────────────────
Symptoms: Spoke-A cannot reach Spoke-B through Hub
Cause:    Missing UDR, Firewall rule, or IP Forwarding

CHECKLIST:
  □ UDR created for each Spoke pointing to NVA IP
  □ UDR associated with correct Subnet
  □ NVA has IP Forwarding enabled on NIC
  □ Firewall/NVA has rules to allow spoke traffic
  □ Peering configured from both Hub→Spoke directions

Check IP Forwarding on Firewall NIC:
  Portal → NVA VM → Networking → NIC → IP Configurations
  □ Enable IP Forwarding = Yes

ISSUE 5: NSG Rule Not Working
─────────────────────────────────────────────────────────────
Symptoms: Traffic blocked despite having allow rule
Cause:    Priority conflict or wrong direction

Solution:
  1. Check Priority: Lower number = higher priority
  2. Verify Direction: Inbound vs Outbound
  3. Check if Deny rule has lower priority number
  4. Verify Source and Destination IP/CIDR are correct
  5. Use "Effective Security Rules" to see applied rules:
     VM → Networking → NIC → Effective Security Rules

DIAGNOSTIC TOOLS:
─────────────────────────────────────────────────────────────

┌────────────────────────────────────────────────────────┐
│  Tool                    │  Use Case                   │
├────────────────────────────────────────────────────────┤
│  IP Flow Verify          │  Test if traffic allowed    │
│  Next Hop               │  Verify routing path        │
│  Connection Monitor      │  Test end-to-end connect    │
│  Effective Security Rules│  See all applied NSG rules  │
│  Effective Routes        │  See all routing entries    │
│  VPN Troubleshoot        │  Debug VPN gateway issues   │
└────────────────────────────────────────────────────────┘
```

---

# Summary and Reference

## Architecture Decision Guide

```
CHOOSING THE RIGHT ARCHITECTURE:
─────────────────────────────────────────────────────────────

SCENARIO → RECOMMENDATION

Simple web + database? 
→ 2-Tier in single VNet

Web + API + Database? 
→ 3-Tier in single VNet

Multiple teams/environments needing isolation? 
→ Multiple VNets with Peering

Many VNets needing central management? 
→ Hub-Spoke with Azure Firewall

Multi-region with global reach? 
→ Azure Virtual WAN

On-premises to cloud connectivity? 
→ Hub VNet with VPN Gateway or ExpressRoute
```

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│                    QUICK REFERENCE                          │
├─────────────────────────────────────────────────────────────┤
│  CONCEPT              │  KEY POINTS                         │
├───────────────────────┼─────────────────────────────────────┤
│  OSI Layer 3          │  IP Addressing, Routing             │
│  OSI Layer 4          │  TCP/UDP, Port Numbers              │
│  VLAN                 │  L2 Segmentation (→ Subnets in Az) │
│  VNet                 │  Azure's private network            │
│  Subnet               │  Subdivisions within VNet           │
│  NSG                  │  L3/L4 firewall for Azure           │
│  VNet Peering         │  Connect two VNets privately        │
│  Peering = NOT Trans  │  A↔B, B↔C ≠ A↔C                   │
│  Hub-Spoke            │  Central VNet routes all traffic   │
│  UDR                  │  Override Azure default routes      │
│  Azure Firewall       │  Managed NVA for hub-spoke          │
│  2-Tier               │  Web + Database                     │
│  3-Tier               │  Web + App + Database               │
├───────────────────────┼─────────────────────────────────────┤
│  Azure Reserved IPs   │  5 per subnet (.0,.1,.2,.3,.255)   │
│  Peering Requirement  │  Non-overlapping CIDRs              │
│  Peering Direction    │  Both sides must be configured      │
│  NSG Processing       │  Subnet NSG → NIC NSG (Inbound)    │
│  NSG Priority         │  100-4096 (lower = higher priority) │
└───────────────────────┴─────────────────────────────────────┘
```

---

*Documentation Version: 1.0*  
*Last Updated: 2024*  
*Platform: Microsoft Azure*  
*Difficulty: Intermediate*