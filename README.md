# Enterprise Network Design, Segmentation and Security Lab

A portfolio-grade enterprise network designed and secured using Cisco Packet Tracer.

This project demonstrates practical networking and network-security concepts including VLAN segmentation, inter-VLAN routing, DHCP, DNS, HTTP, ACLs, NAT/PAT, DMZ architecture, SSH hardening, switch port security, and structured network troubleshooting.

---

## Project Overview

The objective of this project was to design a segmented enterprise network and implement security controls that restrict unnecessary communication while maintaining required connectivity.

The network was built incrementally and tested after each major configuration stage.

The project also includes deliberate failure scenarios to demonstrate a structured troubleshooting methodology.

---

## Network Architecture

The network consists of:

- Employee VLAN
- Server VLAN
- Guest VLAN
- DMZ VLAN
- Management VLAN
- Router-on-a-Stick inter-VLAN routing
- Simulated ISP/Internet
- NAT/PAT
- Static NAT for DMZ web access
- Centralized DHCP
- Internal DNS
- HTTP services
- SSH-based device management

### High-Level Architecture

```text
                         INTERNET
                            |
                           ISP
                            |
                       10.0.0.0/30
                            |
                           R1
                            |
                       802.1Q TRUNK
                            |
                         SW1
          ________________|________________
         |          |          |           |
      VLAN 10    VLAN 20    VLAN 30     VLAN 40
     Employees   Servers     Guest        DMZ
         |          |          |           |
        PCs       Server0      PC3      DMZ Server

                         VLAN 99
                       Management
                           |
                    Management PC
```
## VLAN Design

|  VLAN  |   Purpose    |     Network     | Gateway      |
|--------|--------------|-----------------|--------------|
|   10   |   Employees  | 192.168.10.0/24 | 192.168.10.1 |
|   20   |   Servers    | 192.168.20.0/24 | 192.168.20.1 |
|   30   |   Guest      | 192.168.30.0/24 | 192.168.30.1 |
|   40   |   DMZ        | 192.168.40.0/24 | 192.168.40.1 |
|   99   |   Management | 192.168.99.0/24 | 192.168.99.1 |

## WAN

|       Device      | Interface |    Address    |
|-------------------|-----------|---------------|
|         R1        |    G0/1   | 10.0.0.1/30   |
|         ISP       |    G0/0   | 10.0.0.2/30   |
|         ISP       |    G0/1   | 192.0.2.1/24  |
|   Internet Server |    NIC    | 192.0.2.10/24 |

## Network Services 

#### DHCP

R1 provides DHCP services for:
- VLAN 10
- VLAN 20
- VLAN 30
Addresses .1 through .20 are reserved for infrastructure/static devices.

#### DNS

The internal DNS server is:
192.168.20.2
Example DNS record:server.company.local → 192.168.20.2

#### HTTP

HTTP services are provided by:
Server0
192.168.20.2 
The DMZ web server:
DMZ Server
192.168.40.2

## Security Controls

#### VLAN Segmentation
Different network roles are isolated using separate VLANs.

VLAN 10 → Employees
VLAN 20 → Servers
VLAN 30 → Guest
VLAN 40 → DMZ
VLAN 99 → Management

This reduces the broadcast domain and provides a foundation for access-control policies.

#### Guest Network Isolation
Guests are prevented from accessing the internal server network.

Policy:
Guest → Server VLAN      DENY
Guest → Internet         ALLOW

ACL:
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
permit ip any any

#### DMZ Isolation
The DMZ is separated from internal networks.

Policy:
DMZ → Employee VLAN     DENY
DMZ → Server VLAN       DENY
DMZ → Internet          ALLOW
This demonstrates the basic security principle of placing externally accessible services into a controlled network segment.

#### NAT/PAT
Private internal addresses are translated through R1 before accessing the simulated Internet.

PAT uses the R1 WAN interface:
10.0.0.1

Internal networks:
```
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
```
#### DMZ Static NAT
The DMZ web server is published through R1 using static NAT
```
192.168.40.2:80
        ↓
10.0.0.1:80
```
This allows the simulated external network to access the DMZ HTTP service without exposing the internal server VLAN.

#### SSH Management Security
Network-device management is restricted to VLAN 99.

Management network:
192.168.99.0/24
Only the management network is permitted to establish SSH sessions with:
R1
SW1
Telnet is disabled.

#### Switch Port Security
Port security was configured on access ports using:

Maximum MAC addresses: 1
Sticky MAC learning
Violation mode: Restrict
This prevents unauthorized devices from freely being connected to protected access ports.

## Troubleshooting Methodology 

The project includes deliberate failures to demonstrate practical troubleshooting.

The methodology used was:

```
SYMPTOM
   ↓
HYPOTHESIS
   ↓
TEST
   ↓
EVIDENCE
   ↓
ROOT CAUSE
   ↓
FIX
   ↓
VERIFY
   ↓
DOCUMENT
```
### Failure Scenarios 

Seven failures were deliberately introduced and resolved.

#### 01 — DNS Failure

Cause:
Incorrect DNS server configuration.

Verification:
ping server.company.local
Direct IP connectivity remained functional, proving the network path was operational.

#### 02 — DHCP Failure

Cause:
VLAN 10 DHCP pool was removed.

Result:

Clients failed to obtain an IP address after forcing a new DHCP request.
The DHCP pool was restored and connectivity verified.

#### 03 — VLAN/Trunk Failure

Cause:
The switch-to-router interface was changed from trunk mode to access mode.

Verification:
show interfaces trunk

No active trunk was displayed.
The interface was restored to trunk mode.

#### 04 — ACL Failure

Cause:
An incorrect ACL rule was introduced.
This demonstrated the importance of ACL processing order because ACL entries are evaluated using first-match logic.
The incorrect rule was removed and the intended security policy restored.

#### 05 — NAT Failure

Cause:
The PAT rule was deliberately removed.

Result:
Internal hosts could no longer reach the simulated Internet.

Verification:
show running-config | include ip nat
show ip nat translations
PAT was restored and connectivity returned.

#### 06 — Port Security Failure

Cause:
A different MAC address was connected to a protected switch port.

Result:
Security violation counters increased.

Verification:
show port-security interface fa0/1
The violation was identified and the original topology restored.

#### 07 — Default Gateway / Routing Failure

Cause:
The default route on R1 was removed.

Verification:
show ip route

The router reported:
Gateway of last resort is not set

The default route was restored:
ip route 0.0.0.0 0.0.0.0 10.0.0.2

Internet connectivity was verified again.

## Verification

The following security and connectivity tests were successfully performed:

|          Test           | Expected Result |
|-------------------------|-----------------|
| Guest → Server VLAN     |      BLOCKED    |
| Guest → Internet        |      ALLOWED    |
| DMZ → Server VLAN       |      BLOCKED    |
| DMZ → Internet          |      ALLOWED    |
| Management PC → SW1 SSH |      ALLOWED    |
| Management PC → R1 SSH  |      ALLOWED    |

## Technologies Used 
```
-- Cisco Packet Tracer
-- Cisco IOS
-- VLANs
-- 802.1Q
-- Router-on-a-Stick
-- DHCP
-- DNS
-- HTTP
-- Extended ACLs
-- NAT/PAT
-- Static NAT
-- SSH
-- Port Security
-- Network Troubleshooting
```

## Project Files

The repository contains:

topology/
documentation/
configurations/
evidence/
screenshots/

The original Cisco Packet Tracer project is included so the network can be opened and inspected directly.

## Learning Outcomes 

This project provided practical experience with:
```
-- Enterprise network segmentation
-- Inter-VLAN routing
-- Network services
-- Access-control policies
-- NAT and address translation
-- DMZ architecture
-- Secure network-device management
-- Layer 2 security
-- Network troubleshooting
-- Failure analysis
-- Security validation
```

The project was designed not only to demonstrate configuration ability, but also to demonstrate the ability to identify failures, collect evidence, determine root cause, implement a fix, and verify the result.

  
