# Network Architecture

## Overview

This project implements a segmented enterprise network using Cisco Packet Tracer.

The architecture separates users, servers, guests, externally exposed services, and network management into different VLANs.

Inter-VLAN communication is handled by Router R1 using Router-on-a-Stick.

R1 also provides DHCP services, NAT/PAT, access-control policies, and secure remote management.

---

## Physical Architecture

The main network devices are:

- R1 - Cisco 2911 Router
- SW1 - Cisco 2960 Switch
- ISP - Cisco 2911 Router
- Employee PCs
- Server0
- Guest PC
- DMZ Server
- Management PC
- Internet Server

The primary LAN connection is:

R1 → 802.1Q Trunk → SW1

The WAN connection is:

R1 → ISP → Internet Server

---

## Logical Architecture

```text
                         INTERNET
                            |
                     Internet Server
                       192.0.2.10
                            |
                        ISP Router
                            |
                       10.0.0.0/30
                            |
                       R1 Router
                            |
                       802.1Q Trunk
                            |
                         SW1
             ______________|_______________
            |       |       |       |      |
         VLAN 10 VLAN 20 VLAN 30 VLAN 40 VLAN 99
         Employee Servers  Guest    DMZ   Management
            |       |       |       |      |
           PCs    Server0   PC3   DMZ     Mgmt PC
                                  Server
```
## Router-on-a-Stick

R1 performs inter-VLAN routing using subinterfaces on GigabitEthernet0/0.

Each VLAN is assigned its own subinterface and gateway.

| Interface | VLAN |   Gateway    |
|-----------|------|--------------|
|  G0/0.10  |  10  | 192.168.10.1 |
|  G0/0.20  |  20  | 192.168.20.1 |
|  G0/0.30  |  30  | 192.168.30.1 |
|  G0/0.40  |  40  | 192.168.40.1 |
|  G0/0.99  |  99  | 192.168.99.1 |

The physical G0/0 interface operates as the trunk connection to SW1.

## Switch Architecture

SW1 provides Layer 2 connectivity between endpoint devices and R1.
Access ports are assigned to the appropriate VLAN based on the role of the connected device.
The R1-facing GigabitEthernet interface operates as an 802.1Q trunk.

### Access VLANs
```
Fa0/1 - Fa0/2  → VLAN 10
Fa0/3          → VLAN 20
Fa0/5          → VLAN 30
Fa0/6          → VLAN 40
Fa0/7          → VLAN 99
```
Unused access ports were administratively shut down as part of the switch-hardening configuration.

### WAN Architecture 

R1 connects to the simulated ISP using a /30 point-to-point network.
```
R1 G0/1
10.0.0.1/30
     |
     |
10.0.0.2/30
ISP G0/0
```
The ISP connects to the simulated Internet network:
```
ISP G0/1
192.0.2.1/24
     |
     |
Internet Server
192.0.2.10/24
```
R1 uses a default route pointing toward the ISP.

### Network Segmentation 

The network is divided into five logical security zones.

#### VLAN 10 - Employees

Network: 192.168.10.0/24
Gateway: 192.168.10.1

Used for employee/client systems.

#### VLAN 20 - Servers 

Network: 192.168.20.0/24
Gateway: 192.168.20.1

Server0 provides internal DNS and HTTP services.

#### VLAN 30 - Guest 

Network: 192.168.30.0/24
Gateway: 192.168.30.1

Guest systems are isolated from the internal server network using an extended ACL.
Internet access remains permitted.

#### VLAN 40 - DMZ 

Network: 192.168.40.0/24
Gateway: 192.168.40.1
The DMZ contains the externally accessible web server.
The DMZ is restricted from directly accessing the Employee and Server VLANs.

#### VLAN 99 - Management

Network: 192.168.99.0/24
Gateway: 192.168.99.1

This VLAN is dedicated to network-device management.
SSH access to R1 and SW1 is restricted to this network.

### Traffic Flow

#### Employee → Server
```
PC
 ↓
VLAN 10
 ↓
SW1
 ↓
802.1Q Trunk
 ↓
R1 G0/0.10
 ↓
R1 Routing
 ↓
R1 G0/0.20
 ↓
VLAN 20
 ↓
Server0
```
This demonstrates inter-VLAN routing.

#### Guest → Internal Server
```
Guest PC
 ↓
VLAN 30
 ↓
R1 G0/0.30
 ↓
GUEST-RESTRICTIONS ACL
 ↓
BLOCK
```
Guest access to the internal Server VLAN is intentionally denied.

#### Guest → Internet
```
Guest PC
 ↓
VLAN 30
 ↓
R1
 ↓
NAT/PAT
 ↓
R1 G0/1
 ↓
ISP
 ↓
Internet Server
```
Guest Internet access remains permitted.

#### DMZ → Internal Network

Traffic originating from the DMZ toward internal Employee and Server networks is blocked by the DMZ ACL.

#### Management → Network Devices
```
Management PC
192.168.99.10
       |
       ↓
VLAN 99
       |
       ↓
SW1 / R1
       |
      SSH
```
SSH access is restricted to the Management VLAN.

### Design Principles
The architecture follows several basic enterprise networking and security principles:
- Network segmentation
- Least-privilege access
- Separation of management traffic
- Isolation of guest systems
- DMZ-based placement of externally accessible services
- Controlled Internet access
- Secure remote administration
- Layer 2 access-port protection
- Structured troubleshooting and validation

### Summary 

The resulting architecture provides a small but realistic enterprise environment that demonstrates both networking and security concepts.

The design separates network roles while maintaining controlled communication between required zones.

Security controls were validated through connectivity tests and deliberate failure scenarios.







