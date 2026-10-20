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
