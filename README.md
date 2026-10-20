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
```
|  VLAN  |   Purpose    |     Network     | Gateway      |
|--------|--------------|-----------------|--------------|
|   10   |   Employees  | 192.168.10.0/24 | 192.168.10.1 |
|   20   |   Servers    | 192.168.20.0/24 | 192.168.20.1 |
|   30   |   Guest      | 192.168.30.0/24 | 192.168.30.1 |
|   40   |   DMZ        | 192.168.40.0/24 | 192.168.40.1 |
|   99   |   Management | 192.168.99.0/24 | 192.168.99.1 |
```
## WAN
```
|       Device      | Interface |    Address    |
|-------------------|-----------|---------------|
|         R1        |    G0/1   | 10.0.0.1/30   |
|         ISP       |    G0/0   | 10.0.0.2/30   |
|         ISP       |    G0/1   | 192.0.2.1/24  |
|   Internet Server |    NIC    | 192.0.2.10/24 |
```
## Network Services 

DHCP

R1 provides DHCP services for:
- VLAN 10
- VLAN 20
- VLAN 30
Addresses .1 through .20 are reserved for infrastructure/static devices.

DNS

The internal DNS server is:
192.168.20.2
Example DNS record:
server.company.local → 192.168.20.2

HTTP

HTTP services are provided by:
Server0
192.168.20.2
and the DMZ web server:
DMZ Server
192.168.40.2

